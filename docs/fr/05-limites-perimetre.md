# Chapitre 5 -- Limites et perimetre de ce parcours

Ce parcours couvre le concept central de Base Verify (jetons deterministes
pour la resistance Sybil), la construction du message SIWE encodant
fournisseur, traits et action sous forme d URNs, la validation cote serveur
des traits (protection contre la falsification frontend), le flux
hors-chaine complet avec stockage Prisma et double contrainte d unicite, et
la variante on-chain avec jeton EIP-712 et deduplication par smart
contract.

Sont volontairement laisses hors champ : le detail de l implementation du
contrat `SybilResistantAirdrop` lui-meme (le depot n en contient que
l ABI minimale consommee cote client, pas le code source Solidity), le flux
OAuth complet gere par l application web Base Verify externe
(`verify.base.dev`), la variante `pages/coinbase.tsx` (tres proche de
`pages/index.tsx` mais ciblant specifiquement le fournisseur Coinbase), et
la documentation detaillee presente dans le dossier `/docs` du depot
(`core-concepts.md`, `integration.md`, `traits.md`, `api.md`,
`security.md`), qui constitue deja une reference exhaustive ecrite par
Base elle-meme et n a pas besoin d etre reformulee ici.

L objectif de ce parcours est de comprendre precisement comment une
application consomme l API Base Verify dans ses deux modes d integration
(hors-chaine et on-chain), pas de redocumenter l integralite du guide
officiel deja fourni par le depot.
