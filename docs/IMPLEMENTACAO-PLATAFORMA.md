# Orientações para colocar o curso na plataforma ITA

Este documento orienta a próxima sessão do Claude (ou dev) a importar o curso
**"Formação em Inteligência Artificial Aplicada à Educação" (120h)** para a
plataforma. O conteúdo estruturado está em `docs/curso-ia-educacao-120h.json`
e o documento completo (com fundamentação teórica e referências por sessão)
está no doc colaborativo: https://claude.ai/artifact/MjmfxAEHZVNRGAuhCDzN14

Leia primeiro `.claude/skills/synced/.../ceitec-ita-system/SKILL.md` (arquitetura
completa do ecossistema ITA) antes de tocar em qualquer código. Este arquivo
resume só o que é específico para a implementação do curso.

## 1. Antes de implementar — identificar o módulo certo

Este repositório (`ita-demo`) contém hoje apenas uma landing page estática
(`index.html`). Ele **não** é o código-fonte da plataforma ITA em produção.
Antes de importar o curso:

1. Localizar o repositório real do módulo que vai hospedar cursos/trilhas
   (provavelmente GamificaEdu ou um módulo novo de "Formação de Professores" —
   **não** o Corretor de Provas, que é exclusivamente para provas/gabaritos).
2. Ler o código-fonte desse módulo para descobrir a stack real (Django, Node,
   outra). **Nunca assumir Django** fora do Corretor de Provas — é a regra nº 1
   da arquitetura ITA.
3. Confirmar se já existe um modelo de dados para "curso"/"trilha"/"treinamento".
   Se existir, mapear os campos abaixo para ele em vez de criar um novo.

## 2. Modelo de dados sugerido (caso não exista um)

```
Curso
  - nome, carga_horaria_total, modalidade, publico_alvo, pre_requisitos
  - periodo_inicio, periodo_fim
  - escola (FK)  ← isolamento obrigatório por escola, como em todo o sistema

Modulo
  - curso (FK)
  - numero, titulo, formato (presencial|ead), carga_horaria
  - fundamentacao_teorica (texto + lista de links)

Sessao
  - modulo (FK)
  - codigo (ex.: "2.3"), titulo, carga_horaria
  - objetivo, conteudo (lista), metodologia, exemplo_pratico
  - recursos (lista), avaliacao_produto, referencias (lista)

Matricula
  - cursista (FK usuário), curso (FK), escola (FK)
  - progresso_pct, frequencia_pct, nota_final, status

EntregaAtividade
  - matricula (FK), sessao (FK)
  - arquivo/texto entregue, nota, feedback, timestamp
```

Popular `Curso`, `Modulo` e `Sessao` a partir de `curso-ia-educacao-120h.json`
(estrutura 1:1 com o modelo acima).

## 3. Regras de arquitetura que se aplicam aqui

- **Autenticação:** todo acesso ao curso passa pelo SSO do CEITEC ID (link
  mágico). Nunca criar tela de login própria para o curso.
- **Perfis:**
  - `ita_admin`: cria/edita o curso, vê matrículas de todas as escolas.
  - `coordenador`: matricula professores da própria escola, vê progresso
    apenas da sua escola.
  - `professor`: cursista — vê e avança apenas sua própria matrícula/progresso.
- **Isolamento por escola:** matrículas, progresso e entregas de uma escola
  nunca podem aparecer para outra escola. Se o módulo for Django, aplicar o
  mesmo padrão de `EscolaMixin` usado no Corretor de Provas.
- **Se o módulo escolhido for o Corretor de Provas (Django):** seguir os
  decorators existentes (`@requer_ita_admin`, `@requer_coordenador`,
  `@requer_professor`) e registrar `LogAcessoNegado` em tentativas indevidas.
- Comentar todo o código em português brasileiro (padrão do projeto).

## 4. Sugestão de import

1. Criar migration/schema para `Curso`, `Modulo`, `Sessao` (ou equivalente).
2. Escrever um management command / script de seed que lê
   `curso-ia-educacao-120h.json` e popula as tabelas (idempotente — rodar de
   novo não deve duplicar).
3. Expor o curso na área do professor (`/professor/dashboard/` ou painel
   equivalente) como uma trilha com progresso por módulo.
4. Ligar o `avaliacao` do JSON (pesos: prova presencial 40%, atividades 20%,
   participação 10%, projeto final 30%; frequência mínima 75%; nota mínima
   6,0) à lógica de aprovação/certificação.

## 5. Checklist final (herdado da skill ceitec-ita-system)

- [ ] Identifiquei em qual módulo real da plataforma isso vai morar?
- [ ] Confirmei a tecnologia lendo o código-fonte (não assumi Django)?
- [ ] Uso SSO do CEITEC ID, sem login próprio?
- [ ] Apliquei isolamento por escola nas matrículas/progresso?
- [ ] Os grupos usados são `ita_admin`, `coordenador`, `professor`?
- [ ] Nenhuma funcionalidade existente foi quebrada?
- [ ] Código comentado em português brasileiro?

Se qualquer item acima for "não", parar e corrigir antes de prosseguir.
