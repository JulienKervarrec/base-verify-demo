# Chapitre 2 -- Encoder provider, traits et action dans un message SIWE

Toute la logique de construction du message a verifier est centralisee dans
`lib/signature-generator.ts`, fonction `buildSIWEMessage`. Plutot que
d envoyer les exigences de verification separement de la signature (ce qui
permettrait a un utilisateur malveillant de les modifier avant de les
transmettre au backend), l application les encode directement dans le champ
`resources` du message SIWE (RFC 3986, tel que defini par le standard
EIP-4361) sous forme d URNs (Uniform Resource Names) : le message lui-meme,
une fois signe, devient la preuve infalsifiable de ce que l utilisateur a
accepte de faire verifier.

Le format des URNs suit un schema fixe :
`urn:verify:provider:{provider}:{trait_name}:{operation}:{value}`. Une URN
de base declare le fournisseur (`urn:verify:provider:x`), et des URNs
additionnelles ajoutent des exigences de traits, chacune avec un operateur
de comparaison parmi `eq`, `gt`, `gte`, `lt`, `lte`, `in` (par exemple
`urn:verify:provider:x:followers:gte:1000` signifie "compte X avec au moins
1000 abonnes"). Une URN separee, `urn:verify:action:{action}`, identifie
l action metier concernee (par exemple `claim_airdrop`) : le meme compte
verifie produit des jetons differents pour des actions differentes, ce qui
evite qu un jeton obtenu pour "rejoindre une allowlist" soit reutilisable
pour "reclamer un airdrop" ailleurs.

`generateSignature` orchestre l ensemble : elle accepte soit une cle privee
brute (utile pour des scripts de test), soit une paire adresse plus
fonction de signature fournie par un wallet connecte (le cas normal en
production), construit le message via `buildSIWEMessage`, le fait signer,
et retourne l adresse, le message et la signature prets a etre envoyes au
backend. Une couche de cache (`lib/signatureCache.ts`) memorise la
signature en `localStorage` pendant 5 minutes pour eviter de redemander une
signature au wallet a chaque etape du flux de verification.
