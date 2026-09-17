# Projeto TDE: VozDoCliente IA (Análise de Feedbacks)

**Alunos:** Rijkaard de Sousa de Andrade, Diego Paim, Erik nogueira, Rafael
**Disciplina:** Computação em Nuvem (Unifan 2026.2)

---

## 1. Visão Geral e Diferenciais do Projeto
O **VozDoCliente IA** é uma plataforma baseada em nuvem para capturar, processar e analisar feedbacks de usuários em tempo real. Utilizando arquitetura Serverless e APIs de Inteligência Artificial, o sistema classifica automaticamente a percepção do cliente (Positivo, Negativo ou Neutro).

**O Grande Diferencial:** Além da análise de sentimento padrão, o nosso sistema integra a API do Google Gemini para gerar automaticamente uma **"Sugestão de Plano de Ação"** direcionada ao gestor, transformando reclamações ou elogios em tarefas acionáveis e estratégicas de forma imediata.

## 2. Jornadas de Usuário e Usabilidade
A usabilidade foi pensada para ser fluida e sem gargalos de carregamento:
1. **Envio (Cliente):** O cliente acessa a interface web estática (HTML/JS) e envia seu feedback. O formulário é limpo imediatamente, sem travamentos (processamento assíncrono).
2. **Processamento (Backend):** O frontend dispara uma requisição POST para a nossa API Serverless.
3. **Análise (IA):** A API Serverless empacota o texto e aciona o Google Gemini, solicitando o sentimento e o plano de ação.
4. **Armazenamento:** A resposta estruturada é salva no banco de dados NoSQL (Firebase Firestore).
5. **Consumo (Gestor):** O gestor acessa um Dashboard que escuta o Firestore em tempo real (Real-time listener), visualizando os novos feedbacks e os planos de ação gerados pela IA instantaneamente.

## 3. System Design e Arquitetura Cloud (Diagramas)

Optamos por uma arquitetura PaaS/Serverless focada em Custo Zero, utilizando Vercel e Firebase.

### Arquitetura de Componentes

graph TD
    A[Cliente / Interface Web] -->|HTTP POST| B(Vercel: Serverless Function - Python)
    B -->|REST API| C{Google Gemini API}
    C -->|Retorna Sentimento e Ação| B
    B -->|Grava Documento| D[(Firebase Firestore NoSQL)]
    E[Dashboard do Gestor] -->|Leitura Real-time| D

Fluxo de CI/CD (GitHub Actions)

sequenceDiagram
    participant Dev as Desenvolvedor
    participant Git as GitHub
    participant CI as GitHub Actions
    participant Cloud as Vercel / Firebase

    Dev->>Git: Git Push (branch main)
    Git->>CI: Trigger do Pipeline Automático
    CI->>CI: Executa Testes e Lint
    CI->>Cloud: Deploy Automático
    Cloud-->>Dev: Aplicação Atualizada em Produção

4. Segurança e Gestão de AcessosGestão de Secrets (Least Privilege): Nenhuma chave de API (Google Gemini) ou credencial de banco de dados é exposta no frontend. Elas ficam isoladas como Variáveis de Ambiente (.env) gerenciadas pelos Secrets da Vercel.Regras de Banco de Dados (Firestore Security Rules): O banco de dados bloqueia operações de gravação não autenticadas vindas da internet (allow write: if false;). Apenas o backend Serverless (usando o SDK Admin com credenciais de serviço) possui permissão para gravar os dados.Proteção de Borda: Utilização do rate-limiting nativo da plataforma de hospedagem para mitigar abusos e evitar o esgotamento da cota gratuita da API do Gemini.

5. Planejamento de Escalonamento (Auto Scaling)A grande vantagem da escolha de uma arquitetura 100% Event-Driven e Serverless é que o escalonamento horizontal é gerenciado pelo provedor de nuvem:Scale-out (Picos de uso): Se houver um aumento massivo de feedbacks simultâneos, a plataforma Vercel provisiona novos containers da função Python paralelamente, sem necessidade de configurar Auto Scaling Groups de EC2.Scale-in (Scale-to-zero): Em momentos de ociosidade (madrugada, por exemplo), os recursos computacionais são reduzidos a zero, garantindo que a aplicação não consuma cotas ou gere custos desnecessários.

6. Estrutura de Banco de Dados (Firestore NoSQL)Utilizaremos o modelo baseado em documentos para flexibilidade e velocidade de leitura.Coleção: feedbacks_analisadosid: String (Gerado automaticamente)cliente_nome: Stringfeedback_original: Stringsentimento: String (Enum: POSITIVO, NEGATIVO, NEUTRO)plano_de_acao: String (Gerado pelo Gemini)data_envio: Timestamp

7. Detalhamento de Custos Mensais (Planejamento)O projeto foi rigorosamente desenhado para se manter dentro dos limites do Free Tier (Nível Gratuito) das plataformas escolhidas.

Serviço / Recurso	Função na Arquitetura	Plano Utilizado	Custo (USD)
Vercel Hosting	Hospedagem do Frontend estático	Hobby Tier (Gratuito)	$0.00
Vercel Functions	API Backend Serverless (Python)	Hobby Tier (100k req/mês)	$0.00
Firebase Firestore	Banco de Dados NoSQL	Spark Plan (1GB, 50k leituras/dia)	$0.00
Google Gemini API	Processamento de IA e NLP	Free Tier (Limites de RPM padrão)	$0.00
GitHub	Repositório, CI/CD e Secrets	Free Plan	$0.00
Total Mensal Estimado	Ambiente completo de produção	-	$0.00
