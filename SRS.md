# 📋 Especificação de Produto: Agiliza Aí (v1.4 – Gold Master Final) — Versão Simplificada

## 1. Glossário e Definições (palavras simples)
* **Tenant:** Prefeitura que contratou o sistema (ex: Prefeitura de Ourinhos).
* **Cluster de Recorrência:** Grupo de 3 ou mais problemas da mesma categoria num raio de 50m e em até 7 dias.
* **Geofencing (Raio de Segurança):** Regra que bloqueia um envio se o GPS do celular estiver a mais de 200 metros do local marcado no mapa.
* **GPS Accuracy:** Medida que indica o quão preciso o GPS do celular está (em metros).
* **Incidente Pai:** Caso principal criado quando vários reports viram um cluster.
* **Incidente Filho:** Cada publicação individual do cidadão, que pode fazer parte de um Incidente Pai.
* **Soft Hide:** Esconder temporariamente o conteúdo e mostrar “Em análise” até o gestor decidir.
* **Auto-Hide:** Esconder automaticamente o conteúdo por análise técnica (ex.: imagem com conteúdo impróprio).

---

## 2. Regras de Negócio (RN) — explicadas de forma direta

### 2.1 Identidade e Acesso

| ID | Regra | Explicação |
| :--- | :--- | :--- |
| **RN-001** | **Unicidade de Acesso** | Cada usuário precisa de e-mail ou login social único. No perfil público só aparece o nome e a data de entrada. |
| **RN-002** | **Limite de Flood** | Cada usuário pode criar até 5 publicações em 24 horas (janela móvel). |
| **RN-003** | **Geofencing Obrigatório** | O envio só é permitido se o GPS do celular estiver até 200m do pino no mapa. |
| **RN-004** | **Classificação de Vínculo** | Ao reportar, o usuário escolhe “Residente” ou “Turista” (para estatísticas). |
| **RN-005** | **Direito ao Esquecimento** | Se o usuário apagar a conta, seus dados pessoais são removidos, mas as publicações ficam como “Cidadão Anônimo”. |

---

### 2.2 Ciclo de Vida e Gestão

| ID | Regra | Explicação |
| :--- | :--- | :--- |
| **RN-006** | **Clusterização Automática** | Quando aparecem 3 reports iguais na mesma área e em até 7 dias, o sistema cria um Incidente Pai. |
| **RN-007** | **Resolução em Cascata** | Se o gestor resolve o Incidente Pai, todos os Incidentes Filhos também ficam resolvidos automaticamente. |
| **RN-008** | **Contestação e Desvinculação** | O cidadão pode reabrir o chamado em até 15 dias. Se fizer parte de um cluster, ele é removido do cluster e vira um caso individual. |
| **RN-009** | **Limite de Contestação** | Só é permitida 1 reabertura por chamado. A segunda vez que o gestor fechar é definitiva. |
| **RN-010** | **Edição e Exclusão** | Usuário não pode editar publicação. Pode excluir só se o status for ABERTA. |
| **RN-011** | **Congelamento de Interação** | Chamados resolvidos há mais de 30 dias não aceitam novos comentários. Denúncias ainda são permitidas. |

---

### 2.3 Moderação e Segurança

| ID | Regra | Explicação |
| :--- | :--- | :--- |
| **RN-012** | **Denúncia de Comentários** | Se 3 pessoas diferentes denunciarem um comentário, ele é escondido automaticamente (Soft Hide). |
| **RN-013** | **Denúncia de Publicações** | Também é possível denunciar a publicação inteira. Com 3 denúncias ela entra em Soft Hide. |
| **RN-014** | **Triagem Automática de Mídia** | Imagens são checadas por um serviço automático (ML). Se o risco for muito alto (≥ 0.95) a imagem é escondida imediatamente. |
| **RN-015** | **Proteção contra Abuso de Denúncias** | Um usuário só pode denunciar o mesmo item uma vez. Existem limites por hora para evitar ataques. |
| **RN-016** | **Retenção Segura da Mídia Original** | A imagem original (com EXIF) fica guardada num lugar seguro por até 90 dias para investigação, só moderadores têm acesso. |
| **RN-020** | **Detecção de Denúncia Maliciosa** | O sistema monitora padrões de denúncias inválidas. Se um usuário fizer várias denúncias que forem restauradas pelo gestor em curto período, ele pode ter o recurso de denúncia temporariamente limitado (ação progressiva: aviso → bloqueio 24h → bloqueio 7 dias → suspensão), tudo registrado em auditoria. |

---

### 2.4 Engajamento e Relevância

| ID | Regra | Explicação |
| :--- | :--- | :--- |
| **RN-017** | **Cálculo de Relevância** | Pontos = (Likes × 0.5) + (Comentários × 1.0) + (Compartilhamentos × 1.5). Se for cluster, multiplica por 2. |
| **RN-018** | **Ordenação do Feed (MVP)** | O feed mostra os posts por maior pontuação. Se dois posts tiverem pontuação muito parecida (diferença ≤ 5%), mostramos o mais novo primeiro. |
| **RN-019** | **Notificações** | Mudança de status e ações de moderação enviam push e ficam salvas na central de notificações do app. |

---

## 3. Histórias de Usuário e Critérios de Aceite

### Épico 01 – Reporte de Problemas (App Cidadão)

---

### **US-01 — Criar Publicação com Validação Geográfica**

**Como** cidadão  
**Quero** reportar um problema na rua  
**Para** a prefeitura tomar providências

#### Critérios de Aceite

1. **Reporte com sucesso**
   * Usuário autenticado, GPS com precisão ≤ 50m e dentro de 200m do pino → consegue enviar o chamado e ele vira ABERTA.

2. **Bloqueio por distância**
   * Se estiver a mais de 200m do pino, o app não deixa enviar e mostra o motivo.

3. **Bloqueio por baixa precisão**
   * Se o GPS estiver impreciso (> 50m), o app pede para melhorar o sinal (bloqueia o envio).

4. **Compressão de imagem**
   * Imagem grande (>12MB) é reduzida no app para ≤2MB antes de enviar.

---

### **US-02 — Acompanhamento e Histórico**

**Como** cidadão  
**Quero** ver meus pedidos  
**Para** saber o que está acontecendo

#### Critérios de Aceite

1. Ver lista "Minhas Atividades" com ícones/labels de status claros  
2. Posso reabrir o chamado em até 15 dias  
3. Se o item fazia parte de um cluster, o app mostra aviso que ele ficará individual ao reabrir

---

### **US-03 — Denunciar Comentários**

**Como** usuário  
**Quero** denunciar comentários ruins  
**Para** manter o espaço seguro

#### Critérios de Aceite

1. Ao receber 3 denúncias diferentes, o comentário some (Soft Hide) e vai para o painel do gestor

---

### **US-03b — Denunciar Publicação**

**Como** usuário  
**Quero** denunciar uma publicação inteira (texto ou foto)  
**Para** evitar conteúdo impróprio

#### Critérios de Aceite

1. 3 denúncias diferentes → publicação entra em Soft Hide (texto vira "Em análise", imagem escondida)  
2. Gestor pode Restaurar (volta ao normal) ou Excluir (é removida) — excluir precisa de motivo

---

### Épico 02 – Gestão (Painel do Gestor)


### **US-04 — Triagem e Auditoria**

1. Gestor vê fila de denúncias com evidências (quem denunciou, ML score, etc.)  
2. Ao Restaurar, as denúncias são zeradas  
3. Ao Excluir, o gestor deve explicar o motivo

---

### **US-05 — Gestão de Clusters**

1. Quando o gestor resolve o Incidente Pai, todos os filhos também ficam resolvidos  
2. A nota de conclusão vai para todos automaticamente

---

## 4. Requisitos Não Funcionais (RNF) 

| ID | O que é | Explicação |
| :--- | :--- | :--- |
| **RNF-01** | Segurança | Proteger o sistema contra XSS, SQL Injection e CSRF. |
| **RNF-02** | Privacidade | Tirar dados EXIF das imagens que aparecem para todo mundo. |
| **RNF-03** | Performance | O app reduz imagens no celular antes de enviar (tamanho máximo 1920×1080). |
| **RNF-04** | Concorrência | Criar clusters com cuidado para não duplicar (trava ou job idempotente). |
| **RNF-05** | Disponibilidade | Se o mapa der problema, mostrar uma lista alternativa. |
| **RNF-06** | Acessibilidade | Seguir WCAG 2.1 AA (contraste, leitores de tela, botões grandes). |
| **RNF-07** | Auditoria | Todas as ações de moderação ficam registradas (quem fez, quando e por quê). |
| **RNF-08** | Rate-limit | Limitar número de denúncias e publicações por usuário e por IP. |
| **RNF-09** | Armazenamento | Guardar imagens originais em local seguro por até 90 dias. |

---

📌 **Observações simples**
- Decaimento temporal da relevância (fazer posts antigos perderem prioridade) sai na versão v2.  
- Permitir edição de publicação pode ser avaliado na v1.1 (janela curta).  
- Limitar o número de imagens por publicação fica no backlog (recomendado: máximo 5).

---
