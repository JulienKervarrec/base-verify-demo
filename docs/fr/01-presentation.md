# Chapitre 1 -- Presentation de base-verify-demo

Ce depot est une application Next.js de demonstration officielle de Base
pour "Base Verify" : un service qui permet a un utilisateur de prouver la
possession d un compte verifie sur une plateforme tierce (X/Twitter,
Coinbase, Instagram, TikTok) sans jamais partager ses identifiants avec
l application, et sans que celle-ci n ait a implementer elle-meme un flux
OAuth complet. En echange de cette preuve, l application recoit un jeton
deterministe : le meme compte social produit toujours le meme jeton, ce qui
permet une resistance aux attaques Sybil (un utilisateur ne peut pas
reclamer plusieurs fois un airdrop en connectant plusieurs wallets, tant
qu il utilise le meme compte social pour se verifier).

Le cas d usage demonstre est une reclamation d airdrop : l utilisateur
connecte un wallet, signe un message SIWE (Sign-In with Ethereum) qui
encode a la fois le fournisseur d identite choisi et les exigences
d attributs ("traits", par exemple "compte certifie" ou "1000 abonnes
minimum"), puis l application interroge l API Base Verify avec cette
signature. Si l utilisateur a deja verifie ce compte, l API renvoie un
jeton ; sinon, l utilisateur est redirige vers l application web Base
Verify pour completer le flux OAuth, puis revient terminer la verification.

Le depot va plus loin qu un simple exemple : il propose deux implementations
completes et paralleles du meme concept -- une verification "off-chain"
(`pages/index.tsx`, `pages/coinbase.tsx`) qui stocke le jeton dans une base
de donnees PostgreSQL via Prisma pour empecher les doublons, et une
verification "on-chain" (`pages/onchain.tsx`) ou le jeton est un objet
signe EIP-712 verifie et deduplique directement par un smart contract sur
Base Sepolia. Ce contraste entre les deux approches est un des interets
pedagogiques majeurs de ce parcours.
