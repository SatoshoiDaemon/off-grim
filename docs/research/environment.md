# Ambiente de pesquisa

## Escopo e reprodutibilidade

Análise estática realizada em Windows 11, no diretório C:\Users\axiom\Desktop\grimm-soul, sobre Grim+Soul_+Dark+Survival+RPG_8.6.0_APKPure.xapk. O pacote original foi mantido intacto. O jogo não foi instalado nem executado e nenhuma chamada foi feita a serviços do jogo, lojas ou provedores de pagamento.

Os extratores e o AssetRipper leram cópias em artifacts\. Esse diretório contém material derivado do APK e ferramentas grandes; ele fica fora do Git. Os documentos registram hashes, caminhos, símbolos, evidências REA e limitações para permitir a reprodução sem incluir o APK nem seus assets no histórico.

## Ferramentas verificadas

| Ferramenta | Versão / caminho | Uso e resultado |
|---|---|---|
| REA | 6.10.1, MCP local | Inventário de XAPK/APK, análise Android, metadados IL2CPP e inspeção ELF. Evidências retidas no bundle REA. |
| Java | JDK 17.0.12 em C:\Program Files\Java\jdk-17 | Runtime inicial e ferramentas Android. |
| Eclipse Temurin | JDK 21.0.13.1+2 em %LOCALAPPDATA%\GrimSoul-Research\tools\temurin-21\jdk-21.0.13.1+2 | Runtime usado pelo Ghidra 12.1.4. SHA-256 do ZIP: bbebea1469996b276683c9299c227b294f85c847039c82633218186a4ad4fbf2. |
| Ghidra | 12.1.4 em artifacts\tools\ghidra_12.1.4_PUBLIC | Com JDK 21, importou libil2cpp.so e salvou o projeto headless. A análise automática atingiu 1.200 s; um script Java aplicou labels e scripts dirigidos desassemblaram/descompilaram wrappers e state machines selecionados usando os RVAs do metadata. O pseudocódigo tem globals sem tipo e cobertura parcial. |
| Ghidra | 12.2 em artifacts\tools\ghidra_12.2_PUBLIC | Testado com JDK 25. O framework OSGi/Felix encerrou com The data file must be inside the data dir / dataFile null, inclusive após trocar a pasta de projeto. |
| JADX headless MCP | 0.7.1, %LOCALAPPDATA%\GrimSoul-Research\tools\jadx-headless-mcp-0.7.1-all.jar | Busca e análise de classes DEX. O servidor MCP REA não herdou a variável de configuração; foi usado o cliente local configurado. SHA-256: 6e5eacf500b64292bfb73c49797c1958f6ee44646e43e868039ae7feb573ff75. |
| Apktool | 3.0.3, %LOCALAPPDATA%\GrimSoul-Research\tools\apktool-3.0.3\apktool.bat | Decodificação do manifesto e recursos do APK base. SHA-256 do JAR: dbf930b076c6b9be08d57c449cacefc3bdd6b71ebd59b3066fc0e1f5b14f9423. |
| Android SDK Build Tools | 37.0.0; apksigner 0.9 | Verificação criptográfica das assinaturas dos três APKs. |
| AssetRipper | 2.0.0, %LOCALAPPDATA%\GrimSoul-Research\tools\assetripper-2.0.0\AssetRipper.GUI.Free.exe | Importação headless dos arquivos Unity extraídos, por serviço local em 127.0.0.1. Não executa o jogo. Identificou Unity 6000.3.23f1 e 31.956 objetos em 20 coleções; o log registra dependências Unity não encontradas. |
| Il2CppDumper | 6.7.46, %LOCALAPPDATA%\GrimSoul-Research\tools\il2cppdumper-6.7.46 | Testado contra o binário e metadados. Recusou global-metadata.dat versão 39 (Metadata file supplied is not supported version[39]). |
| Il2CppDumper AndnixSH | 3.2.1, commit 526f7c13cbc33fc7f738e3935a8c4d8085912a8e; CLI local em %LOCALAPPDATA%\GrimSoul-Research\tools\il2cppdumper-cli-3.2.1-win-x64\Il2CppDumper.exe | Build self-contained feita com .NET SDK 10.0.404. SHA-256 do executável: 745EF5B7F0A6212D8B06FFE6B570ED2E6E4118BC7E52EC552EB8498EE92CA3EB. Interpretou metadata v39, encontrou CodeRegistration/MetadataRegistration e gerou dump.cs, 114 DLLs dummy e script.json; emitiu aviso de possível proteção. Não houve bypass nem dump de memória. |
| Python | 3.13.14 | Inventário de ZIP/arquivos e verificação de integridade. |
| Node.js | 26.13 | Runtime disponível para ferramentas REA. |
| Git | 2.56 | Repositório não existia na pasta no início desta fase; nenhum remoto estava configurado. |

## Instalação e configuração

As variáveis de usuário JAVA_HOME, REA_JADX_MCP_JAR e REA_APKTOOL_COMMAND foram configuradas para as ferramentas locais. A distribuição do Ghidra 12.1.4 e o JDK 21 foram baixados de releases oficiais e verificados pelos hashes publicados. AssetRipper foi executado apenas no loopback; o conteúdo do jogo não foi enviado a um serviço remoto.

O adaptador Android de recursos do REA reportou provedor Apktool indisponível porque o MCP já iniciado não herdava REA_APKTOOL_COMMAND. A mesma análise foi realizada pelo apktool.bat local e o resultado JSON foi preservado em artifacts\analysis-evidence\decoded-resources-base.json.

Il2CppDumper 6.7.46 não entende a versão 39 do formato. O fork AndnixSH 3.2.1, compilado do commit registrado acima, completou a ligação estática ARM32 sem extração de memória, mas avisou que o arquivo pode estar protegido. Gerou nomes e RVAs para 221.716 entradas ScriptMethod; o script Java do Ghidra aplicou 186.299 labels e não aplicou 35.417 entradas. Esse total não significa 186 mil funções analisadas. A análise automática de libil2cpp.so expirou em 1.200 s. Depois, scripts Java desassemblaram e decompilaram métodos-alvo por suas entradas metadata, incluindo `StorageController.Sync`, `Load`, `LoadData`, `Request`, `TryPostToken`, `UpdateConfig`, `LoadYookassaUrl`, `PaymentsController.Purchase_Yookassa`, `DarkFileStorage` Load/Save/WriteToFile e `FakeDataServer` PostLoad/PostSave. O pseudocódigo mostra wrappers assíncronos, uma chamada de Sync a Request e file I/O local; ainda contém globals sem tipo, e não é um grafo de chamadas completo nem prova comportamento em runtime.

## Limites de análise dinâmica

O SDK Android encontrado tinha Build Tools, mas não tinha platform-tools\adb.exe, emulador, imagens de sistema nem AVD. Nenhum APK foi instalado ou iniciado. Também não foi criada uma conta de teste ou usada uma conta pessoal. Uma análise dinâmica futura deve usar um AVD descartável com tráfego externo bloqueado desde antes da inicialização; este relatório não contém observação de runtime.

## Fontes das evidências locais

- artifacts\analysis-evidence\android-package-summary.json — componentes e resumo do APK.
- artifacts\analysis-evidence\decoded-resources-base.json — manifesto e recursos decodificados.
- artifacts\analysis-evidence\il2cppdumper.log — incompatibilidade de versão do Il2CppDumper.
- artifacts\analysis-evidence\ghidra-12.1.4-application.log — execução headless com limite de 1.200 s e avisos de instruções sobrepostas.
- artifacts\analysis-evidence\ghidra-targeted-functions.log, `ghidra-method-gaps.log`, `ghidra-load-data-flow.log`, `ghidra-local-storage-flow.log` e `ghidra-offline-fallback-analysis.log` — pseudocódigo dirigido para métodos e state machines relacionados a save/load, API de sessão e providers remoto/local.
- artifacts\analysis-evidence\ghidra-offline-server-xrefs.log — xrefs encontrados para os construtores de providers na base parcial do projeto Ghidra; zero referência encontrada para `FakeDataServer` não demonstra ausência absoluta.
- artifacts\il2cppdumper-output-v39\dump.cs, script.json e DLLs dummy — saída local não versionada do fork compatível com metadata 39.
- artifacts\analysis-evidence\assetripper.log — log de importação do AssetRipper.
- artifacts\analysis-evidence\assetripper-content-inventory.json — versão Unity, coleções, contagens por arquivo e classes de assets.
- Evidências REA exportadas em artifacts\analysis-evidence\rea-evidence-bundle.json.

## Referências oficiais de ferramentas

- [REA](https://github.com/mandiant/REA) e skill local C:\Users\axiom\.agents\skills\reverse-engineer-anything\SKILL.md.
- [Ghidra 12.1.4](https://github.com/NationalSecurityAgency/ghidra/tree/Ghidra_12.1.4_build) e requisitos de Java em GhidraDocs/GettingStarted.md.
- [Apktool](https://github.com/iBotPeaches/Apktool).
- [JADX](https://github.com/skylot/jadx).
- [Il2CppDumper](https://github.com/Perfare/Il2CppDumper).
- [Fork AndnixSH do Il2CppDumper](https://github.com/AndnixSH/Il2CppDumper), usado para metadata versão 39.
- [AssetRipper](https://github.com/AssetRipper/AssetRipper).
