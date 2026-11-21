# Arquitetura identificada

## Conclusão técnica

Grim Soul 8.6.0 é um jogo Unity compilado com IL2CPP, com código de gameplay C# convertido para código nativo e tabelas IL2CPP mantidas no APK. Não é uma build Mono tradicional: o artefato contém libil2cpp.so, libunity.so e global-metadata.dat, e o conjunto principal não apresentou assemblies de gameplay Mono para decompilação.

O pacote contém evidência de bridges e SDKs Android em DEX/Java, mas o fluxo central de gameplay aparece nos tipos App.* do metadata IL2CPP. A engine, portanto, hospeda gameplay no cliente; a existência desse código não prova que o servidor não valida estados ou recompensas.

## Evidências da engine e do código

REA identificou global-metadata.dat versão 39 com 31 seções, 114 imagens, 18.502 tipos, 125.265 métodos, 80.056 campos, 27.450 propriedades, 534 eventos, 110.842 parâmetros e 28.013 literais de string. Os nomes de domínios e métodos seguem legíveis nas tabelas. Isso indica metadata retido suficiente para mapear boa parte do modelo e dos pontos de entrada, apesar de a lógica compilada estar em libil2cpp.so.

Bibliotecas nativas notáveis:

- libil2cpp.so: runtime e código nativo IL2CPP, 2.419 exports inspecionados pelo REA.
- libunity.so: runtime Unity, dependências dinâmicas e imports ELF inspecionados.
- libsigner.so: biblioteca de assinatura/proteção presente; presença não revela sua finalidade completa.
- Bibliotecas de Firebase, publicidade/atribuição, analytics, Bugsnag e libc++_shared.so.

O XAPK fornece somente a ABI armeabi-v7a no split de arquitetura. Isso é uma limitação do arquivo recebido; não prova que nenhuma outra publicação tenha splits adicionais.

O metadata v39 foi ligado estaticamente à biblioteca ARM32 com o fork AndnixSH Il2CppDumper 3.2.1: a ferramenta reconheceu CodeRegistration em 0x4579d44 e MetadataRegistration em 0x4654850 e produziu dump.cs, script.json e 114 assemblies dummy. A saída alertou “This file may be protected”, mas completou o mapeamento; não foi feito bypass nem dump de memória. O AssetRipper identificou exatamente Unity 6000.3.23f1; o detector de versão do metadata mostrou apenas a família 6000.3.0b1+.

## Camada Android

O pacote Android é fantasy.survival.game.rpg; a atividade principal decodificada é androidbridge.com.brickworksgames.unityplugin.DarkPlayerActivity. O inventário REA registrou 106 componentes Android e 32.361 classes DEX, muitas pertencentes a bibliotecas de terceiros. Não se deve interpretar esses números como código próprio do jogo.

O manifesto inclui bridges de Google Play Games/Facebook, Google Ads, Unity Ads, Vungle, AppLovin, IronSource, Firebase/DevToDev, Adjust, faturamento e Bugsnag. São dependências integradas ao cliente. A análise estática não mostra, por si só, quais são obrigatórias para iniciar ou para o loop de gameplay.

## Dados e recursos Unity

O split de dados possui milhares de arquivos com nomes hash e sem extensão, diretório Unity assets/bin/Data, resources, assets, bundles, idiomas, Addressables e imagens de ofertas. AssetRipper 2.0.0 reconheceu Unity 6000.3.23f1, 20 coleções e 31.956 objetos. Há arquivos level1 a level8 e sharedassets1 a sharedassets8; esses nomes descrevem arquivos de dados, não necessariamente áreas jogáveis distintas.

O log do AssetRipper informa dependências não encontradas globalgamemanagers.assets e sharedassets0.assets. A contagem dos objetos carregados é reproduzível, mas a reconstrução semântica de cenas e dependências está incompleta. Não foram exportados projetos Unity nem arquivos proprietários.

## Persistência e domínio do cliente

Tipos próprios identificados no metadata:

| Tipo | Índice do tipo / métodos | Evidência funcional nos nomes |
|---|---:|---|
| App.Data.DarkFileStorage | 3970 / 6 | Load, Save, WriteToFile, Replace, rawData |
| App.Data.DarkRefsStorage | 3984 | Load, Save, refsData |
| App.Data.DarkNetwork | 3977 / 5 | Get, GetInternal, Post, Tick |
| App.Flow.Storage.StorageController | 3687 / 42 | Load, Save, Sync, Request, Receive, LoadRefs, TryPostToken, LoadData |
| App.Core.DarkAppCore | 3994 / 68 | coordena player, map, bank, offer, quests, session, ads e outros modelos |
| App.Core.DarkMigrationService | 4002 / 39 | sequência de migrações locais numeradas até schema 27 |
| App.Core.Users.Player | 4065 / 71 | estado de jogador, equipamentos, inventário, recursos, perks e companheiros |

O binário contém dois níveis de persistência local. `DarkFileStorage` tem `_savePath`, `_prevPath` e `_tempPath`; o estado de `Load` usa `FileStream`, e `WriteToFile` usa `FileStream`/`Stream.WriteAsync`, verifica arquivos e move o save atual para cópia anterior antes de promover o temporário. A leitura de `DarkFileStorage.<WriteToFile>d__12.MoveNext` está no RVA 0x179E4A0. Isso comprova um caminho de arquivo local no cliente, embora o local, a criptografia e o ciclo de vida do save final dependam de mais análise.

Também há a interface `StorageController.IDarkDataServer`, implementada por `DarkDataServer` e `FakeDataServer`. A primeira mantém fila de posts e usa a camada `DarkNetwork`; a segunda tem operações equivalentes de load/save, um arquivo local derivado de `Application.persistentDataPath` e métodos que chamam `File.Exists`, `FileStream`, `File.ReadAllTextAsync` e `Stream.WriteAsync`. O metadata contém `/user_fakedata.json`. Isso demonstra uma implementação local de simulação de serviço no código; não demonstra que a build normal a selecione. O campo `_dataServer` é da interface, mas a análise dirigida não encontrou referência de chamada ao construtor de `FakeDataServer` na base parcial de xrefs do Ghidra. Como a análise global expirou, ausência de referência não prova código morto.

Na camada normal, `DarkDataServer` expõe Post/PostLoad/PostSave e encaminha tarefas ao `DarkServer`/`DarkNetwork`. O Ghidra conseguiu descompilar alguns wrappers e state machines após disassembly dirigido pelas entradas metadata. Um vínculo direto foi confirmado: `StorageController.Sync` (RVA 0x16BD5D0) chama `StorageController.Request` (RVA 0x16BD728). `StorageController.Load` também chama `LoadRefs` e `Run` no fluxo assíncrono que o pseudocódigo recuperou. Esses são vínculos estáticos preliminares; não revelam a resposta de servidor necessária para cada fluxo nem o que acontece durante boot sem rede.

## Vínculo nativo e limites da descompilação

O fork gerou nomes e RVAs; um script Java importou labels no projeto Ghidra 12.1.4 para 186.299 de 221.716 entradas ScriptMethod. As outras 35.417 não foram aplicadas; a causa individual não foi classificada. A análise automática headless expirou após 1.200 s, com avisos de sobreposição de instruções. Scripts Java de análise dirigida desassemblaram entradas metadata e recuperaram pseudocódigo de métodos de StorageController, DarkFileStorage e ambos providers. O pseudocódigo ainda tem globals/ponteiros sem tipo e limites de função foram iniciados nas entradas metadata; não equivale a análise completa do ELF nem prova runtime. Os logs locais registram o trabalho.

RVAs úteis do arquivo `dump.cs` (offsets relativos à base ELF): `StorageController.LoadRefs` 0x16BCA1C; `TryPostToken` 0x16BCB14; `UpdateConfig` 0x16BCC40; `ScheduleServerNotifications` 0x16BCF54; `LoadData` 0x16BD02C; `Sync` 0x16BD5D0; `Request` 0x16BD728; `GetLive` 0x16BD87C; `LoadYookassaUrl` 0x16BD9B0; `LoadXsollaUrl` 0x16BDB04; `PaymentsController.Purchase_Yookassa` 0x16F46A4; `Purchase_Xsolla` 0x16F47EC; `DarkFileStorage.WriteToFile` 0x179C2E8; `Quest.TryComplete` 0x180BBE4; `TakeReward` 0x180BEB8. São associações de metadata para endereços nativos; não provam que os métodos sejam chamados em todos os fluxos nem revelam o corpo por si só.

## Fontes

- artifacts\analysis-evidence\android-package-summary.json.
- artifacts\analysis-evidence\decoded-resources-base.json.
- REA IL2CPP metadata: global-metadata.dat, hash na seção inventário; evidência principal ev_e4a7ab6d2646d983258246a3e08c10c6698bb4b0d3e1716620e3c491b041d9bd.
- Vínculo estático e RVAs: `artifacts\il2cppdumper-output-v39\dump.cs` e `script.json`; pseudocódigo Ghidra em `artifacts\analysis-evidence\ghidra-targeted-functions.log`, `ghidra-local-storage-flow.log`, `ghidra-offline-fallback-analysis.log` e `ghidra-load-data-flow.log`.
- ELF de libil2cpp.so: dependências e imports no REA, evidência principal ev_306d184a94e14778313f3fb246b69039608091b72e121fd9451c93de78e71b3b.
