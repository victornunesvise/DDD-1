# Dinâmica: Design Estratégico do Projeto

## Objetivo
Identificar os subdomínios do projeto, classificá-los (Core, Supporting, Generic) e desenhar os bounded contexts, incluindo suas interações. Esse exercício ajudará a criar uma visão clara e estratégica do domínio.

---

## 1. Nome do Projeto
**TrainHub**

---

## 2. Objetivo Principal do Projeto
TrainHub é um sistema de gestão para academias que centraliza alunos, planos/pagamentos, check-in e treinos, facilitando o controle da operação e a experiência do aluno.

---

## 3. Identificação dos Subdomínios
Liste os subdomínios do sistema e classifique-os como Core Domain, Supporting Subdomain ou Generic Subdomain.

| **Subdomínio**              | **Descrição**                                                                                      | **Tipo**         |
|-----------------------------|--------------------------------------------------------------------------------------------------|------------------|
|  Gestão de Matrículas e Planos  |    Criação/renovação/cancelamento de planos, status do aluno (ativo/inativo), regras de acesso por plano.              | Core Domain      |
|  Controle de Acesso e Check-in  |    Valida se o aluno pode entrar (plano ativo, pendências), registra frequência e controla tentativas de acesso.       | Core Domain      |
|    |                                              | Generic          |

---

## 4. Desenho dos Bounded Contexts
Liste e descreva os bounded contexts identificados no projeto. Explique a responsabilidade de cada um.


| Bounded Context | Responsabilidade | Subdomínios Relacionados |
|---|---|---|
| **Contexto de Matrículas & Planos** | Gerencia o ciclo de vida da matrícula (ativação, renovação, pausa e cancelamento), associação a planos e regras que impactam o acesso do aluno. | Gestão de Matrículas e Planos |
| **Contexto de Acesso & Check-in** | Valida e registra entradas na academia, aplicando regras de acesso (ex.: matrícula ativa, sem inadimplência, restrições de horário) e mantendo histórico de frequência. | Controle de Acesso e Check-in |
| **Contexto de Treinos** | Cria e atribui fichas de treino, organiza treinos e exercícios, define parâmetros (séries, reps, carga sugerida) e controla versões/atualizações de treino. | Gestão de Treinos |
| **Contexto de Avaliações & Evolução** | Registra avaliações físicas e medidas, acompanha progresso e histórico do aluno (indicadores e comparativos ao longo do tempo). | Evolução e Avaliações |
| **Contexto de Aulas & Reservas** | Gerencia aulas coletivas, horários, capacidade, reservas, lista de espera e presença em aula. | Agendamento de Aulas e Reservas |
| **Contexto de Operação & Equipe** | Administra colaboradores, papéis e permissões internas (recepção, instrutor, admin) e rotinas operacionais básicas. | Gestão de Equipe e Operação |
| **Contexto de Cobrança & Pagamentos** | Processa cobranças recorrentes e avulsas, controla faturas, confirmações/recusas, inadimplência e integrações com gateways de pagamento. | Pagamentos e Cobrança |
| **Contexto de Comunicação** | Dispara notificações e mensagens (cobrança, confirmação de matrícula, lembrete de aula, avisos), com templates e canais. | Notificações e Comunicação |
| **Contexto de Relatórios & Indicadores** | Consolida dados e gera métricas (frequência, churn, inadimplência, receita, ocupação de aulas) para visão gerencial. | Relatórios e Indicadores |
| **Contexto de Identidade & Acesso (Auth)** | Gerencia autenticação, sessão, recuperação de senha e políticas de segurança para alunos e equipe. | Autenticação e Segurança |


---

## 5. Comunicação entre os Bounded Contexts
Explique como os bounded contexts vão se comunicar. Use os padrões de comunicação, como:
- **Mensageria/Eventos (desacoplado):** Ex.: O Contexto de Consultas emite um evento "Consulta Finalizada", consumido pelo Contexto de Pagamentos.
- **APIs (síncrono):** Ex.: O Contexto de Pagamentos consulta informações de preços no Contexto de Consultas.

| Origem | Destino | Tipo de Relacionamento | Explicação |
|---|---|---|---|
| Contexto de Matrículas & Planos | Contexto de Cobrança & Pagamentos | Customer–Supplier | Cobrança depende das regras do plano (valor, recorrência, vigência). Matrículas fornece essas regras; Pagamentos “consome” para cobrar corretamente. |
| Contexto de Cobrança & Pagamentos | Contexto de Matrículas & Planos | Customer–Supplier | Matrículas depende do resultado financeiro (fatura paga/vencida) para manter status do aluno (adimplente/inadimplente) e ativar/suspender matrícula. |
| Contexto de Matrículas & Planos | Contexto de Acesso & Check-in | Customer–Supplier | Check-in depende da matrícula estar ativa e das regras de acesso vinculadas ao plano. |
| Contexto de Cobrança & Pagamentos | Contexto de Acesso & Check-in | Customer–Supplier | Check-in precisa saber se o aluno está inadimplente (ou com pagamento confirmado) para permitir/bloquear a entrada. |
| Contexto de Identidade & Acesso (Auth) | Contexto de Matrículas & Planos | Conformist | Matrículas “conforma” com o modelo de identidade (userId, claims, roles) do provedor de autenticação, sem tentar impor mudanças nele. |
| Contexto de Identidade & Acesso (Auth) | Contexto de Operação & Equipe | Conformist | Operação usa as permissões/roles definidas no Auth (admin, recepção, instrutor), aceitando o modelo como vem. |
| Contexto de Operação & Equipe | Contexto de Treinos | Customer–Supplier | Treinos dependem da existência de “Instrutor/Coach” e suas permissões. Operação fornece e gerencia equipe; Treinos consome. |
| Contexto de Matrículas & Planos | Contexto de Aulas & Reservas | Customer–Supplier | Reservas dependem de regras do plano (direito a aulas, limites, elegibilidade). |
| Contexto de Aulas & Reservas | Contexto de Comunicação | Conformist | Comunicação apenas consome eventos/dados de aula (aula marcada/cancelada/lotada) para enviar mensagens, sem influenciar o modelo de aulas. |
| Contexto de Cobrança & Pagamentos | Contexto de Comunicação | Conformist | Comunicação consome eventos de cobrança (fatura gerada, vencimento, pagamento aprovado/falhou) para disparar lembretes e recibos. |
| Contexto de Acesso & Check-in | Contexto de Relatórios & Indicadores | Conformist | Relatórios só consome eventos de check-in para métricas (frequência, horários de pico), aceitando o modelo do check-in como está. |
| Contexto de Treinos | Contexto de Avaliações & Evolução | Partnership | Treinos e Avaliações evoluem juntos: avaliações influenciam progressão e ajustes de treino, e treinos geram dados que alimentam evolução. Requer colaboração e alinhamento constante. |
| Contexto de Matrículas & Planos | Contexto de Treinos | Shared Kernel (mínimo) | Compartilham um núcleo pequeno e estável (ex.: `AlunoId` e/ou “referência do aluno”) para garantir consistência de identificação entre contextos sem acoplar regras de negócio. |
| Gateway de Pagamento (externo) | Contexto de Cobrança & Pagamentos | Anticorruption Layer (ACL) | Pagamentos traduz o modelo do provedor (charge, subscription, webhook) para o modelo interno (Fatura, Pagamento, Inadimplência) sem “vazar” conceitos externos pro domínio. |


---

## 6. Definição da Linguagem Ubíqua
Liste os termos principais da Linguagem Ubíqua do projeto. Explique brevemente cada termo.

| **Termo**                    | **Descrição**                                                                                   |
|------------------------------|-----------------------------------------------------------------------------------------------|
| Ex.: Consulta                | Sessão médica entre paciente e médico.                                                       |
| Ex.: Paciente                | Usuário que agenda e realiza consultas.                                                      |
| Ex.: Receita                 | Prescrição médica gerada durante a consulta.                                                 |

---

## 7. Estratégia de Desenvolvimento
Para cada tipo de subdomínio, explique a abordagem para implementação:
- **Core Domain:** Desenvolver internamente com foco total.
- **Supporting Subdomain:** Desenvolver internamente ou parcialmente terceirizar.
- **Generic Subdomain:** Usar ferramentas ou serviços de mercado.

| **Subdomínio**              | **Estratégia**                         | **Ferramentas ou Serviços (se aplicável)** |
|-----------------------------|---------------------------------------|-------------------------------------------|
| Gestão de Consultas         | Desenvolvimento interno               |                                           |
| Cadastro de Usuários        | Interno com uso de Auth0 para login   | Auth0                                     |
| Pagamentos                  | Terceirizar usando API Stripe         | Stripe                                    |

---

## 8. Diagrama Visual (Opcional, mas Recomendado)
Desenhe um diagrama que mostre:
- Os bounded contexts.
- Como eles se comunicam.
- A relação com os subdomínios.

Use ferramentas como **Miro**, **Lucidchart** ou mesmo papel e caneta para criar seu diagrama e adicionar ao projeto.

---

## Dicas para Apresentação
- Explique cada parte do design, focando no **Core Domain** (o coração do negócio).
- Justifique por que certos subdomínios foram classificados como Supporting ou Generic.
- Destaque como a comunicação entre bounded contexts foi pensada para ser escalável.

---

Boa sorte com a dinâmica! 🚀
