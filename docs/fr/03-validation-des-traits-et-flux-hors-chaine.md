# Chapitre 3 -- Validation des traits cote serveur et flux hors-chaine (base de donnees)

Le point de securite le plus important du depot est `lib/trait-validator.ts`
et sa fonction `validateTraits`. Puisque le message SIWE et sa signature
transitent par le frontend avant d atteindre le backend, rien n empeche
en theorie un utilisateur de modifier le code cote client pour generer un
message reclamant des exigences plus faibles (par exemple "10 abonnes" au
lieu du "1000 abonnes" reellement requis par l application) avant de le
signer. `validateTraits` reparse les URNs du message recu et les compare
strictement aux traits attendus, codes en dur cote serveur dans
`pages/api/verify-token.ts` (`expectedTraits = { verified: 'true' }`) :
toute divergence -- trait manquant, valeur differente, ou trait
supplementaire non attendu -- fait echouer la requete avec un code 400,
avant meme d appeler l API Base Verify.

Une fois cette validation passee, `verify-token.ts` transmet la signature et
le message a l API Base Verify (`POST /base_verify_token`, authentifiee par
une cle secrete jamais exposee au frontend). Trois codes de reponse
structurent le flux : 200 signifie que l utilisateur est verifie et
satisfait les traits (le jeton est extrait et stocke), 404 signifie qu il
n a pas encore verifie ce fournisseur (l application doit alors le rediriger
vers l application web Base Verify via une URL `https://verify.base.dev`),
et 400 signifie qu il est verifie mais ne satisfait pas les traits (erreur
finale, sans interet a reessayer).

Le stockage utilise un modele Prisma `VerifiedUser` avec deux contraintes
d unicite : `address` (une adresse ne peut reclamer qu une fois) et
`baseVerifyToken` (le meme compte social, verifie depuis n importe quel
wallet, produit toujours le meme jeton -- c est cette deuxieme contrainte
qui constitue la protection anti-Sybil reelle). Avant d ecrire, le code
verifie explicitement qu aucun enregistrement existant ne porte deja ce
jeton, renvoyant une erreur 409 sinon. `pages/api/delete-airdrop.ts`
complete ce flux en permettant a un utilisateur de supprimer sa propre
reclamation, en verifiant sa signature sur un message fixe
("Delete airdrop for {address}") avant de retirer son enregistrement --
mais sans jamais liberer le jeton associe a une reutilisation, puisque
reverifier le meme compte social redonnerait de toute facon le meme jeton.
