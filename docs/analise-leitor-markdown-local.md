# Analise Tecnica - Leitor Markdown Local (Mysidian)

## 1. Contexto
Este documento define a analise inicial para o aplicativo local de leitura e edicao de arquivos Markdown do projeto Mysidian.

Objetivo do produto:
- Criar uma versao simples de um app no estilo Obsidian.
- Executar 100% localmente, sem dependencia de backend remoto nesta fase.
- Priorizar curva de aprendizado facil com Node e stack web conhecida.

## 2. Regras Funcionais (fornecidas)
As regras abaixo devem ser implementadas no produto:

1. Ler o arquivo selecionado.
2. Apresentar o conteudo do arquivo selecionado em modo interpretado.
3. Editar o arquivo em modo interpretado, sem expor markdown bruto como experiencia principal.
4. Linkar um arquivo no outro.
5. Organizar arquivos em pastas e hierarquias.

## 3. Requisitos Funcionais Detalhados
Para tornar as regras verificaveis, cada uma foi refinada em comportamento esperado:

### 3.1 Leitura de arquivo selecionado
- O usuario seleciona um arquivo `.md` em uma arvore de diretorios local.
- O app abre e carrega o conteudo em ate 300 ms para arquivos pequenos (ate 200 KB).
- O app deve tratar erros de permissao e arquivo inexistente com mensagem clara.

### 3.2 Renderizacao interpretada
- O conteudo deve ser renderizado em HTML seguro (sem scripts executaveis).
- Suportar recursos basicos de Markdown no MVP:
  - titulos
  - paragrafos
  - listas
  - links
  - bloco de codigo
- Suporte a tabelas nao entra nesta fase; sera tratado em uma fase futura.

### 3.3 Edicao em modo interpretado
- Fase inicial: editor markdown com preview sincronizado (lado a lado).
- Experiencia principal para usuario final: leitura e edicao com feedback visual imediato.
- Evolucao posterior: modo mais visual (WYSIWYG parcial ou completo), se houver necessidade real.

### 3.4 Links entre arquivos
- MVP: suportar links Markdown padrao `[texto](arquivo.md)`.
- O clique no link deve resolver caminho relativo ao arquivo atual.
- Se arquivo de destino nao existir, apresentar apenas uma mensagem de erro.

### 3.5 Pastas e hierarquia
- Exibir arvore de diretorios local.
- Permitir navegacao por pastas e abertura de arquivos por clique.
- Ordenacao inicial recomendada: pastas primeiro, depois arquivos, em ordem alfabetica.

### 3.6 Pasta padrao da aplicacao
- O app possui uma pasta padrao configurada (ex.: `~/Mysidian` ou caminho definido nas preferencias).
- Ao iniciar, o app verifica se a pasta padrao existe no sistema de arquivos.
- Se nao existir, o app a cria automaticamente antes de exibir a interface.
- O usuario pode alterar a pasta padrao nas configuracoes; a nova pasta e criada automaticamente se tambem nao existir.
- A arvore de diretorios e carregada a partir da pasta padrao na inicializacao.

## 4. Requisitos Nao Funcionais
1. Local-first: todo fluxo deve funcionar sem internet.
2. Privacidade: arquivos permanecem no sistema local do usuario.
3. Compatibilidade: Linux (prioridade inicial), com caminho para Windows/macOS.
4. Performance: renderizacao fluida para notas comuns (ate 200 KB).
5. Confiabilidade: evitar corromper arquivo durante salvamento.
6. Seguranca: sanitizar HTML para bloquear XSS no preview.

## 5. Comparativo de Tecnologias Sugeridas
## 5.1 Opcao A - Electron + React + TypeScript
Vantagens:
- Melhor alinhamento com Node e ecossistema JavaScript.
- Curva de aprendizado mais simples para equipes web.
- Grande quantidade de exemplos, pacotes e suporte da comunidade.
- Integracao direta com filesystem local via processo principal.

Desvantagens:
- Binario final maior.
- Consumo de memoria geralmente maior que alternativas nativas.

Quando escolher:
- Quando a prioridade e velocidade de desenvolvimento e menor risco tecnico no inicio.

## 5.2 Opcao B - Tauri + Frontend TypeScript (React/Vue/Svelte)
Vantagens:
- Distribuicao menor e melhor eficiencia de recursos.
- Boa opcao para longo prazo quando performance/tamanho pesam.

Desvantagens:
- Exige lidar com setup Rust no backend nativo.
- Curva de aprendizado operacional maior para quem quer foco total em Node no inicio.

Quando escolher:
- Quando o projeto ja tiver maturidade e necessidade clara de otimizacao de binario e uso de memoria.

## 5.3 Recomendacao para este projeto
Recomendacao em duas etapas:
1. Iniciar com Electron + React + TypeScript para acelerar o MVP local.
2. Reavaliar migracao para Tauri apos validar produto e requisitos reais de performance.

Justificativa:
- Menor atrito para comecar com Node.
- Entrega mais rapida das regras essenciais.
- Reduz risco de travar o projeto em complexidade de infraestrutura cedo demais.

## 6. Bibliotecas Candidatas (sem implementacao ainda)
Parser/render Markdown:
- `remark` + `rehype`
- `markdown-it`
- `marked`

Sanitizacao HTML:
- `DOMPurify`
- `sanitize-html`

Editor com preview:
- `CodeMirror 6`
- `Monaco Editor`
- `Milkdown` (se evoluir para experiencia mais rica)

File watching/arvore:
- `chokidar` para observacao de alteracoes de arquivo/pasta
- `fs` nativo do Node para leitura/escrita

## 7. Regras Executaveis do Sistema
Estas regras definem como o app deve se comportar em runtime:

1. Resolucao de caminhos:
- Sempre resolver links relativos a partir da pasta do arquivo atual.

2. Seguranca de render:
- Nao executar scripts embutidos em markdown/HTML.
- Sanitizar sempre antes de injetar HTML no preview.

3. Politica de links:
- Links relativos internos devem abrir arquivo local.
- Links externos (http/https) devem abrir no navegador do sistema.

4. Sincronizacao editor/preview:
- Toda alteracao no editor atualiza preview em tempo real (com debounce curto, ex. 100-200 ms).

5. Persistencia de arquivo:
- Salvar com escrita atomica quando possivel para reduzir risco de corrupcao.

6. Organizacao de conteudo:
- A arvore reflete a estrutura real de pastas do disco.
- Mudancas externas de arquivo devem ser refletidas no app (watcher).

7. Pasta padrao:
- Na inicializacao, verificar existencia da pasta padrao via `fs.existsSync`.
- Se ausente, criar com `fs.mkdirSync(path, { recursive: true })` antes de qualquer operacao de UI.
- Registrar em log quando a pasta for criada automaticamente.
- Nao sobrescrever pasta existente nem seu conteudo.

## 8. Arquitetura Proposta (alto nivel)
Camadas:
1. UI (janela desktop): arvore de arquivos, area de editor, area de preview.
2. Dominio Markdown: parse, transformacao, sanitizacao e render.
3. Infra local: leitura/escrita de arquivos, watch de diretorio, resolucao de caminhos.

Fluxo principal:
1. Usuario seleciona arquivo na arvore.
2. App le arquivo local.
3. Parser converte markdown em estrutura renderizavel.
4. Sanitizacao gera HTML seguro.
5. Preview exibe conteudo interpretado.
6. Usuario edita e salva, mantendo sincronizacao visual.

## 9. Roadmap de Entrega
## Fase 1 - MVP local
- Definir e criar pasta padrao automaticamente na primeira execucao.
- Abrir pasta local.
- Listar hierarquia de arquivos/pastas.
- Abrir arquivo `.md`.
- Exibir preview interpretado.
- Suportar links markdown padrao entre arquivos.

## Fase 2 - Edicao melhorada
- Editor com preview sincronizado em tempo real.
- Atalhos basicos de produtividade.
- Melhorias de UX para navegacao entre notas.

## Fase 3 - Recursos avancados opcionais
- Busca em conteudo.
- Suporte opcional a wiki links `[[arquivo]]`.
- Evolucao para modo de edicao mais visual (WYSIWYG parcial/completo).

## 10. Fora de Escopo Inicial
- Sincronizacao em nuvem.
- Colaboracao em tempo real.
- Sistema de plugins complexo.

## 11. Riscos Principais e Mitigacoes
Risco: complexidade alta de WYSIWYG completo no inicio.
Mitigacao: iniciar com editor markdown + preview sincronizado.

Risco: vulnerabilidade de XSS em preview HTML.
Mitigacao: sanitizacao obrigatoria e bloqueio de script.

Risco: retrabalho por troca prematura de stack desktop.
Mitigacao: validar produto com Electron antes de avaliar migracao.

## 12. Criterios de Aceite do Planejamento
1. Documento cobre todas as regras solicitadas.
2. Comparativo tecnologico apresenta trade-offs claros.
3. Estrategia local-first esta explicita.
4. Existe roadmap incremental sem dependencia de codigo nesta etapa.

---
Ultima atualizacao: 2026-05-01
Status: planejamento aprovado e documentado para inicio da implementacao.
