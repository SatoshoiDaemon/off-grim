# Dependências de rede

## Evidência que sustenta a análise

O manifesto declara INTERNET, ACCESS_NETWORK_STATE e ACCESS_WIFI_STATE, além de identificadores e atribuição de anúncios. O metadata contém a abstração App.Data.DarkNetwork (Get, GetInternal, Post, Tick) e a camada App.Flow.Storage.StorageController, que combina load/save/sync com token, binding de plataformas, estado live e configuração.

No metadata foram encontrados literais compatíveis com rotas e hosts:

- Rotas: /load, /save, /sync, /token, /friends/token, /user_fakedata.json, /xsolla/payments, /yookassa/payments.
- Telemetria: https://ams-gs02.grimsoulgame.com/stat até ams-gs05.grimsoulgame.com/stat.
- Literais HTTP de tracking: http://46.148.236.125:9800/track e http://5.9.63.14:9800/track.
- Pagamentos: https://secure.xsolla.com/paystation4/?token=.
- Outros: URL de exclusão de conta em brickworksgames.com, sites de política e de distribuição.

As strings foram observadas em global-metadata.dat, mas nesta fase não foram ligadas a xrefs de execução. Endpoints podem estar em código próprio, SDK, código morto ou compatibilidade antiga. Não afirmo que alguma requisição ocorreu, nem tentei contatar esses hosts.

O fork Il2CppDumper produziu RVAs que ligam alguns nomes de método a endereços em `libil2cpp.so`: `StorageController.TryPostToken` 0x16BCB14, `LoadRefs` 0x16BCA1C, `UpdateConfig` 0x16BCC40, `LoadData` 0x16BD02C, `Sync` 0x16BD5D0, `Request` 0x16BD728 e `GetLive` 0x16BD87C. Scripts Ghidra dirigidos recuperaram pseudocódigo preliminar desses pontos. Em `Sync`, uma chamada direta no código nativo resolve para a entrada de `Request`. As state machines de `Load`, `LoadData` e `Request` são assíncronas. O pseudocódigo ainda deixa campos/globals sem tipo e não confirma método HTTP, ordem completa do boot, obrigatoriedade de resposta ou autoridade do servidor.

## Classificação por responsabilidade

| Dependência | Classificação | Evidência e confiança |
|---|---|---|
| Provider de dados de sessão | Híbrido, com alternativa local no código | `StorageController` guarda `IDarkDataServer`; `DarkDataServer` usa fila, `DarkNetwork` e Post; `FakeDataServer` implementa a mesma interface e lê/escreve arquivo local. A base parcial de xrefs não encontrou referência ao construtor do fake provider; seleção no runtime indeterminada. Confiança alta sobre as duas implementações, baixa sobre qual está ativa. |
| Login/token e vínculo de contas | Híbrido / provável backend | StorageController.TryPostToken, BindFacebook, BindApple, BindPlatform, TearPlatform, playerId e rotas /token, /friends/token. Confiança média; não foi observado requisito de boot. |
| Save, load e sincronização | Híbrido | `DarkFileStorage.<WriteToFile>d__12.MoveNext` (RVA 0x179E4A0) usa `FileStream`, `Stream.WriteAsync`, caminhos temporário/anterior e operações Move; `Sync` chama `Request` (0x16BD728), com rotas /load, /save, /sync. `FakeDataServer` oferece load/save local. Confiança alta sobre caminhos presentes; provider ativo e autoridade final incertos. |
| Configuração de jogo / refs | Híbrido ou indeterminado | LoadRefs (0x16BCA1C), UpdateConfig (0x16BCC40), GetLive (0x16BD87C), LoadVersionStatus, refsData; settings Addressables não desabilita atualização. Confiança média; destinos e conteúdo atualizável ainda não estão decodificados. |
| Notificações / temporada / eventos | Híbrido | Scheduler de notificações do servidor, estado live e modelos de oferta/battlepass. Confiança média para funções de live; dependência do loop principal não comprovada. |
| Ads e atribuição | Serviço de terceiros | Google Ads, Unity Ads, Vungle, AppLovin, IronSource, Adjust e Firebase aparecem no pacote; permissões e métodos de anúncios recompensados. Confiança alta sobre integração, uso real em cada sessão desconhecido. |
| Analytics / crash reporting | Serviço de terceiros | Firebase/DevToDev/Adjust e Bugsnag no DEX/native; hosts /stat e tracking como literais. Confiança alta para SDKs integrados, chamadas em runtime não observadas. |
| Compras | Google Play + possíveis provedores externos | BILLING, UnityIAPImpl, PaymentsController, rotas e URLs Xsolla/YooKassa. Confiança alta sobre integração; compra não realizada. |
| Login social / conquistas | Terceiros | Componentes e classes de Facebook/Google Play Games e controllers de plataforma. A funcionalidade offline é indeterminada. |

## Limites e próximo teste seguro

Uma biblioteca de rede, permissão ou host literal não prova que combate, crafting ou IA são server-authoritative. Por outro lado, a existência de classes locais também não prova que o cliente consegue iniciar sem um token ou configuração obtida remotamente.

Nenhum pacote foi executado, nenhuma conta foi usada e nenhum endpoint foi contatado. Para distinguir boot, sessão e salvamento, o próximo teste é uma instalação descartável em emulador com tráfego externo desabilitado e logs locais. A máquina não tinha adb, emulador ou AVD, então esse teste não foi realizado nesta fase.

## Fontes

- Tipos IL2CPP em global-metadata.dat; ver architecture.md.
- Manifesto decodificado em artifacts\analysis-evidence\decoded-resources-base.json.
- Componentes DEX/Android em artifacts\analysis-evidence\android-package-summary.json.
- Literais de metadata são estáticos; não há PCAP, logcat ou captura dinâmica associada.
