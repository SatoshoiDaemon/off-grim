# Inventário do artefato

## Original e integridade

Arquivo de entrada preservado:

C:\Users\axiom\Desktop\grimm-soul\Grim+Soul_+Dark+Survival+RPG_8.6.0_APKPure.xapk

| Medida | Resultado |
|---|---|
| Tamanho do XAPK | 716.544.077 bytes |
| SHA-256 | C470AFF6D8766F09AE3A93680704C1CB237AD6B2222CF13827719F8AC32EA75A |
| SHA-1 | 0A222DB7165E832345290BB8648660A00D6CB3AE |
| Formato / inventário | Arquivo ZIP XAPK; 17 de 17 entradas enumeradas sem contradição |
| Diretório de extração | artifacts\extracted-xapk |

SHA-1 e SHA-256 identificam os bytes examinados; não atestam que o distribuidor publicou o arquivo nem que o conteúdo é benigno. O jogo não foi executado.

## Conteúdo do XAPK

manifest.json informa o pacote fantasy.survival.game.rpg, nome Grim Soul, versão 8.6.0, código 806000831, minSdkVersion 25 e targetSdkVersion 36. O manifesto lista três APKs:

| Arquivo | Papel declarado | Tamanho | SHA-256 |
|---|---|---:|---|
| fantasy.survival.game.rpg.apk | base | 56.841.826 | 89692f9226983bf4d3569f2ee50a660cf5757ae059889aa6f291145bb76f3b30 |
| UnityDataAssetPack.apk | UnityDataAssetPack | 623.453.935 | b7927fa24d05440eec230626678fccbe59145e2651cd04f1acb8ac63ba5443f0 |
| config.armeabi_v7a.apk | split de arquitetura ARMv7 | 35.821.009 | 9ee0d4e0eea53562ea3934bd9621f10209036d7cd721e58ce09a7f2966b791c7 |

Não há OBB listado no manifesto XAPK. O conjunto soma 716.116.770 bytes, o total_size declarado no próprio manifesto. A estrutura contém assets e bibliotecas nos splits; a ausência de OBB não significa que todo conteúdo remoto esteja incluído.

## Permissões relevantes

O manifesto do XAPK e o manifesto Android decodificado incluem INTERNET, ACCESS_NETWORK_STATE, ACCESS_WIFI_STATE, com.android.vending.BILLING, com.google.android.gms.permission.AD_ID, permissões de serviços de anúncios/atribuição, notificações, foreground service, wake lock e licença do Google Play. Permissões mostram capacidade solicitada, não provam que cada função foi usada.

## Assinaturas

apksigner 0.9 confirmou assinatura criptográfica válida para os três APKs. O certificado dos APKs base e ARMv7 tem SHA-256:

77b30611d8c92b7c7a60ccce689b8f56449b7b999234914f1ce6312ffb098c94

O Source Stamp reportado tem SHA-256 3257d599a49d2c961a471ca9843f59d341a405884583fc087df4237b733bbd6d, com organização indicada como Brickworks games. O analisador de assinatura do REA não conseguiu processar a entrada de dados Unity por seu limite de 512 MiB; apksigner local fez a verificação dos três splits.

Não foi obtida uma cópia oficial de Grim Soul 8.6.0 para comparar o certificado ou o Source Stamp. Assim, a verificação comprova que os APKs estão assinados e íntegros em relação à assinatura contida neles; não confirma sua origem oficial nem confiabilidade.

A página pública do [Google Play](https://play.google.com/store/apps/details?id=fantasy.survival.game.rpg) identifica o app/publisher, mas não fornece o APK arquivado da versão 8.6.0 nem a fingerprint do certificado daquela versão. A [documentação oficial do Play Console](https://support.google.com/googleplay/android-developer/answer/16641489) descreve a fingerprint como dado obtido para o desenvolvedor no Play Console. Não havia referência pública suficiente para uma comparação criptográfica independente.

## Conteúdo Unity

UnityDataAssetPack.apk passou a verificação CRC do ZIP e foi extraído para artifacts\unity-data com validação de caminhos de destino. A extração produziu 12.183 arquivos e 622.718.389 bytes. Não foram aceitos caminhos escapando da raiz nem links simbólicos.

O inventário por extensão registrou 11.243 arquivos sem extensão, 883 .resource, 16 .json, 12 .bundle, 8 .assets, 6 arquivos .split0/.split1/.split2, mais os arquivos de catálogo e metadados. Isso é um inventário físico, não uma prova de que cada arquivo seja conteúdo acessível em jogo.

AssetRipper 2.0.0 reconheceu a versão de editor Unity 6000.3.23f1 e 20 coleções com 31.956 objetos no diretório Unity extraído. Há arquivos chamados level1 a level8 e sharedassets1 a sharedassets8; os maiores arquivos de cena/coleção por nome são level4 (18.515 objetos), level1 (7.012), level2 (3.910), level7 (1.376), level6 (618) e level8 (463). Estes são nomes de arquivos Unity, não uma contagem de mapas de gameplay.

Durante o processamento, AssetRipper informou dependências ausentes globalgamemanagers.assets e sharedassets0.assets. A contagem é daquilo que foi carregado, não uma reconstrução completa dos objetos que dependem desses arquivos.

assets/aa/settings.json declara Addressables 2.9.1, catálogo local catalog.bin, m_DisableCatalogUpdateOnStart: false e até três requisições simultâneas. catalog.bin tem 8.136 bytes e catalog.hash tem 32 bytes. O catálogo é suficiente para reconhecer uma configuração de conteúdo atualizável, mas sua decodificação completa e os destinos efetivos ainda precisam ser confirmados. A opção de atualização não prova que uma atualização remota foi executada.

Outros achados: refs.bin (12.697.046 bytes), treze arquivos JSON de idioma e doze bundles com nomes smartoffers_assets_image_offer_*. Essa distribuição favorece uma hipótese de pacote híbrido: assets locais e possibilidade de atualização remota.

## Arquivos nativos e dados locais

O pacote inclui libil2cpp.so (76.146.660 bytes), libunity.so (17.295.932 bytes), libsigner.so e bibliotecas de Firebase/analytics, publicidade, Bugsnag e C++. As bibliotecas enumeradas estão na arquitetura armeabi-v7a; não foi encontrado split arm64-v8a, x86 ou x86_64 no XAPK recebido.

global-metadata.dat está em assets/bin/Data/Managed/Metadata/, mede 13.679.884 bytes e tem SHA-256 8a13d6271f915d57520e25db23c32e4de62756b3bd4a7c25aa91682d2f3314e1.

Busca por arquivos .db, .sqlite, .sqlite3 ou .realm nos arquivos extraídos não encontrou correspondências. Isso não exclui saves em arquivos sem extensão, armazenamento serializado, SharedPreferences criado em runtime ou bancos gerados no dispositivo.

## Evidências para reprodução

- XAPK, hashes e estrutura: REA inspect_artifact / inventory, evidências ev_1fb4a55729a04b47c7ef93c21c616c221a3a530d2f6bf57563cb4ac5cf4d45ec e ev_fcd7d3ed7349fe1e80b0440f8144197dac36bc6f44aa2760ac3f50a01afeea8b.
- Extração da estrutura: REA extract_artifact, evidência ev_4b7cd9f36c965c892b05f70813bd16c3ec9c2444e04a8b60fb6170c331923542.
- APK base / recursos: artifacts\analysis-evidence\decoded-resources-base.json, APK SHA-256 89692f9226983bf4d3569f2ee50a660cf5757ae059889aa6f291145bb76f3b30.
- Inventário semântico parcial Unity: artifacts\analysis-evidence\assetripper-content-inventory.json e log artifacts\analysis-evidence\assetripper.log.
