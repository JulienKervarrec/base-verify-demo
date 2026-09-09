# Chapitre 4 -- La variante on-chain : jeton EIP-712 et deduplication par smart contract

`pages/onchain.tsx` et `pages/api/onchain/verify-token.ts` implementent une
approche alternative qui elimine completement la base de donnees. Au lieu
d appeler `/base_verify_token`, cette route appelle
`/onchain_verify_token` sur l API Base Verify, en lui passant en plus
l adresse du contrat cible (`config.claimContractAddress`, un
`SybilResistantAirdrop` deploye sur Base Sepolia). La reponse n est plus un
simple jeton opaque a stocker en base, mais un objet signe EIP-712 a six
champs (`owner`, `target`, `action`, `uniqueHash`, `expiration`,
`delegate`, `signature`) directement exploitable comme parametre d appel de
contrat.

Le frontend appelle ensuite `claim(uniqueHash, expiration, delegate,
signature)` sur ce contrat via `useWriteContract` de wagmi. Le contrat lui-
meme est responsable de la deduplication : il maintient un mapping
`claimed[uniqueHash]` et refuse toute reclamation dont le hash a deja ete
consomme (erreur `AlreadyClaimed`), verifie que le jeton n est pas expire
(`TokenExpired`), et que la signature provient bien d un signataire de
confiance (`UntrustedSigner`). Le `uniqueHash` etant derive de facon
deterministe du compte social verifie (de la meme maniere que le jeton
hors-chaine), la meme garantie anti-Sybil s applique, mais l etat de
deduplication vit entierement on-chain plutot que dans une base de donnees
geree par l application -- avec pour consequence qu aucune ecriture
serveur n est necessaire cote application ("No DB write: dedup lives
on-chain", comme le documente un commentaire du code source), au prix
d une transaction on-chain a la charge de l utilisateur (ou d un relayeur)
pour chaque reclamation.

Le composant gere egalement un pre-check en lecture seule
(`publicClient.readContract` sur `claimed`) avant de proposer la
transaction, afin d avertir l utilisateur en amont si ce hash a deja ete
reclame, evitant une transaction vouee a l echec.
