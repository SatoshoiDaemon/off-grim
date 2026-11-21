# Decisão técnica: Plano A ou Plano B

## Recomendação

**Prosseguir primeiro com o Plano A em um protótipo local, privado e sem rede.** Além dos modelos e operações de jogador, inventário, combate, quests, mapas, crafting e economia, o binário contém escrita de save local e duas implementações da interface de dados: uma com rede (`DarkDataServer`) e uma local com arquivo (`FakeDataServer`). O `Sync` chama `Request`, enquanto o provider local implementa load/save. Isso melhora a perspectiva de adaptar o cliente sem recriar o gameplay; ainda é necessário provar que o provider local pode ser conectado ao fluxo da build normal.

Essa recomendação é condicional. Ainda não foi demonstrado que a tela de jogo ou uma sessão utilizável inicia sem token/configuração remota, nem que todo conteúdo relevante está no APK. O Plano A não deve ser considerado concluído ou plenamente viável até que um teste em ambiente isolado mostre inicialização, progressão e salvamento offline.

## Comparação

| Critério | Plano A — adaptar o cliente original | Plano B — reconstrução independente |
|---|---|---|
| Reuso do gameplay | Maior: regras e estado já existem em IL2CPP e metadata | Nenhum reuso de implementação; recriar regras e IA |
| Conteúdo visual/dados | Grande pacote Unity disponível no artefato local, sujeito aos direitos aplicáveis | Criar arte, sons, mapas e dados próprios |
| Serviço que precisa ser substituído | Auth/token, API de save/sync, refs/config, funções live, loja, ads e analytics | Criar toda a infraestrutura local necessária, sem backend |
| Principal risco | Fluxos centrais podem depender de bootstrap, assinatura de save ou dados remotos; manutenção de binário IL2CPP é difícil | Escopo de implementação e produção de conteúdo é muito maior; fidelidade ao original é baixa no começo |
| Complexidade provável | Alta a muito alta | Muito alta |
| Prazo para um protótipo offline | Menor se o cliente aceitar armazenamento local e configuração incluída | Maior; exige sistema de jogo jogável independente |
| Propriedade intelectual | Exige trabalho local sobre o artefato e cuidado para não redistribuir código/assets proprietários | Requer código, assets e nomes próprios; usar o relatório apenas como referência técnica |

## Plano A — gates para provar viabilidade

1. Preparar emulador descartável, instalar o XAPK validado e bloquear tráfego externo antes de iniciar.
2. Observar a sequência offline de inicialização sem criar conta nem tentar autenticação remota.
3. Confirmar se o cliente aceita DarkFileStorage sem resposta de StorageController ou se aguarda token, versão, refs ou live config.
4. Localizar o formato e o ciclo de vida do save criado localmente, usando apenas um perfil de teste vazio.
5. Rastrear quem atribui `StorageController._dataServer` e confirmar se `FakeDataServer` é selecionável; o Ghidra não achou xref para seu construtor no banco parcial, mas essa ausência não é conclusiva.
6. Continuar a análise de corpos e xrefs já ligados por RVA; wrappers/state machines selecionados foram decompilados, mas o grafo global do Ghidra expirou em 1.200 s e há campos globais sem tipo.
7. Inspecionar catálogo Addressables e confirmar presença dos dados para campanha, mapas, receitas, inimigos e recompensas.
8. Substituir login/sync por perfil local e storage, remover integrações de compra/ads na implementação privada e reequilibrar recompensas para gameplay.
9. Confirmar gameplay, salvamento, encerramento e reabertura com rede bloqueada. Um build ou uma tela de menu não bastam para validar a experiência.

Se a sessão offline exigir resposta assinada de servidor, configuração remota obrigatória ou validação constante de estado, revisar o gate do Plano A. Isso não prova impossibilidade, mas aumenta o custo e a fragilidade da adaptação.

## Plano B — quando passa a ser melhor

Escolher a reconstrução somente se os gates mostrarem que a execução offline depende estruturalmente de serviços indisponíveis, se o binário não puder ser mantido com segurança ou se a meta exigir um produto que não use componentes proprietários. O conhecimento útil para a reconstrução está no mapa de domínio e nas dependências deste relatório; a implementação deve usar código e recursos próprios e não presumir direito de redistribuir os arquivos extraídos.

## Decisão desta fase

Fase 1 confirma a presença de lógica local extensa e de forte camada híbrida de serviços. Portanto, a decisão fundamentada é **Plano A como primeiro protótipo, com confiança média**, não uma declaração de que o jogo completo já funciona offline. O Plano B permanece uma alternativa autorizada caso autenticação, sync ou controle remoto sejam um bloqueio estrutural.

## Evidências e limitações

Base: inventário XAPK e assinaturas em artifact-inventory.md, arquitetura e RVAs em architecture.md, mapa de sistemas em gameplay-systems.md, literais e integrações em network-dependencies.md e classificação resumida em offline-feasibility.md. A análise foi estática; wrappers/state machines selecionados foram decompilados, incluindo Sync→Request e file I/O do save local, mas a seleção do provider e os fluxos completos de sessão/pagamento não estão determinados. Nenhum pagamento, conta, servidor ou dado de terceiro foi acessado.
