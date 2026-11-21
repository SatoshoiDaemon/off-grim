# Viabilidade offline

## Resumo

Há lógica e dados de domínio substanciais no cliente: player, inventário, combate, quests, mapas, crafting/construção, IA, economia e armazenamento local. O pacote também contém um grande split Unity e metadata suficiente para mapear milhares de tipos. Isso favorece um protótipo offline baseado no cliente original.

Ainda não há prova de que a experiência completa inicie sem token, configuração, catálogo, refs ou save carregados de servidor. O projeto é tecnicamente promissor para o Plano A, mas a viabilidade integral continua condicionada à inicialização e a chamadas de persistência/autorização não observadas.

O vínculo dirigido de parte do binário fortalece a hipótese de adaptação: `DarkFileStorage` grava por arquivo com temporário/cópia anterior, `StorageController.Sync` chama `Request`, e existe um `FakeDataServer` que implementa a interface de dados usando arquivo local. A seleção desse provider local pela build normal não foi encontrada na análise parcial de xrefs; por isso, ainda não é evidência de modo offline ativo.

## Matriz por sistema

| Sistema | Categoria | Evidência estática | Confiança | Trabalho para offline |
|---|---|---|---|---|
| Cálculo de combate no cliente | Local | ArmorMath, DamageResolve, DamagePipeline em metadata | Média-alta | Confirmar por pseudocódigo e executar localmente; drops e resultado podem ter validação externa. |
| Inventário e operações de item | Local | Inventory, InventorySet, AddStack, RemoveStack, Split, Exchange | Alta para lógica local; média para persistência | Preservar regras locais e ligar ao save local. |
| Estado do jogador e equipamento | Híbrido | Player, DarkAppCore.player, storage local e Sync | Alta | Perfil local, IDs locais e merge/migração de save. |
| IA/inimigos e mundo explorável | Local ou híbrido | tipos de inimigo/AI, Location, Dungeon, MapMovement | Média | Confirmar tabelas e inicialização; eventos/seed podem vir do backend. |
| Quests, crafting e base | Local ou híbrido | Quest methods, receitas e modelos de construção no metadata | Média | Inventariar dados Addressables/refs; garantir recompensas e receitas sem serviço. |
| Save/load | Híbrido, persistência local confirmada no código | DarkFileStorage WriteToFile MoveNext (RVA 0x179E4A0) usa FileStream/WriteAsync e promove arquivo temporário; StorageController.Sync (0x16BD5D0) chama Request (0x16BD728); FakeDataServer fornece load/save local | Alta para implementação local/híbrida | Descobrir formato e obrigatoriedade no boot; avaliar se o provider local pode ser selecionado sem alterar o jogo nesta fase. |
| Identidade/autenticação | Híbrido ou indeterminado; provider local disponível no código | StorageController.TryPostToken (RVA 0x16BCB14), social bindings, playerId, rotas token; FakeDataServer tem PostLoad/bind operations, mas seu uso normal não foi provado | Média | Confirmar requisito de inicialização e qual implementação atende as operações sem serviço. |
| Configuração e balanceamento | Indeterminado / híbrido | UpdateConfig (0x16BCC40), LoadRefs (0x16BCA1C), GetLive (0x16BD87C), Addressables com atualização habilitada | Média | Determinar quais dados de refs/catálogo são obrigatórios e manter cópia local. |
| Eventos live, notificações e social | Dependente de serviço para a função live | métodos de server notification, Facebook/Google Play Games | Média-alta para integrações; baixa para requisito de boot | Desligar ou substituir eventos online; o conteúdo de campanha ainda precisa ser inspecionado. |
| Loja, assinatura e compra | Dependente de serviço externo | IAP, Billing, Xsolla/YooKassa, métodos de compra | Alta | Remover interface comercial e reequilibrar progressão com fontes de gameplay. |
| Anúncios recompensados | Dependente de serviço, com efeito no gameplay | SDKs de ads e RewardedVideo / AdsController | Alta | Substituir recompensas e acelerações por sistemas de gameplay locais. |
| Analytics e crash reporting | Serviço externo opcional/indeterminado | SDKs e hosts como literais | Média | Remover ou desativar chamadas na build privada offline. |

## Quanto conteúdo existe no pacote

O split de dados Unity mede 623.453.935 bytes. Após extração validada, foram 12.183 arquivos (622.718.389 bytes), com catálogo Addressables, recursos, assets, bundles, idioma e imagens de ofertas. AssetRipper identificou Unity 6000.3.23f1 e carregou 31.956 objetos em 20 coleções, incluindo arquivos level1 a level8. Isso demonstra uma base grande de conteúdo instalada. Não demonstra que todo mapa, evento, inimigo, campanha, tabela de drop ou configuração esteja incluído; Addressables também permite atualização. O log do AssetRipper registra dependências ausentes, portanto a cobertura semântica permanece incompleta.

## Conclusões para as perguntas da fase

1. **Arquitetura:** Unity + IL2CPP, código de domínio C# convertido em ARM32 nativo, camada Android e serviços/SDKs externos.
2. **Sistemas fundamentais locais:** há classes para personagem, combate, inventário, quests, mapas, inimigos, crafting/build e persistência.
3. **Dependências de servidor:** autenticação/token, save/sync, refs/config, conteúdo live e possivelmente eventos/recompensas. Não se conhece a obrigatoriedade de cada um no boot.
4. **Conteúdo presente:** centenas de MB e milhares de arquivos estão no XAPK, incluindo catálogo, assets, recursos e idiomas; cobertura total do jogo não está provada.
5. **Adaptar o original offline:** tecnicamente viável em princípio, com risco alto na inicialização, autenticação e sincronização; ainda não demonstrado em execução.
6. **Código próprio provável:** storage e identidade locais, configurações/catálogo offline, adaptadores de serviços, regra econômica sem compras/ads e substituição de notificações/eventos online.
7. **Complexidade:** Plano A alta a muito alta; Plano B muito alta, pois teria de recriar sistemas e conteúdo em código/arte próprios.
8. **Estratégia:** priorizar um protótipo privado do Plano A; manter o Plano B como alternativa se as APIs de bootstrap e autoridade forem inseparáveis do servidor ou se o cliente não puder ser mantido de forma estável.

## Lacunas que podem mudar a conclusão

- Nenhum teste de instalação ou de boot offline foi realizado; não havia ADB/emulador no host.
- As rotas são literais de metadata, não uma captura de tráfego nem xrefs confirmadas.
- O formato de save e o fluxo completo de recuperação de sessão são desconhecidos.
- Catálogo e tabelas de Addressables não foram decodificados até cobertura completa.
- O fork AndnixSH Il2CppDumper 3.2.1 ligou metadata v39 a RVAs; Ghidra importou 186.299 labels e recuperou pseudocódigo dirigido, mas a análise automática expirou em 1.200 s e o grafo global/xrefs continua incompleto. A ausência de referências ao construtor de FakeDataServer no estado parcial não prova que ele nunca seja ativado.
- Assinatura válida foi verificada, mas não houve comparação com publicação oficial.
