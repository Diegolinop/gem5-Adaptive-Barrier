# Validação das correções — 9 de novembro de 2026

Referência: TA2, capítulo 11, especialmente os critérios de aceitação da p. 408
e as seções 11.8–11.14. A numeração é a impressa no livro.

## Executado

- 108 casos nativos comparados com as referências sequenciais: micro, Jacobi e
  integração; centralizada, disseminação e auto; P=1,2,3,4,8,16; 3 e 4 iterações.
- Verificações independentes: Jacobi 3x3 com bordas 100 resulta em soma 900;
  integração de 4/(1+x*x) em [0,1], com 10.000 subintervalos, aproxima pi com
  erro absoluto inferior a 1e-8.
- 45 execuções gem5 TimingSimpleCPU: três programas, três algoritmos,
  P=1,2,4,8,16, quatro iterações, trabalho/tamanho 19, semente 1, limiar de teste 4.
  Todas passaram na comparação de resultado, conservação de episódios/chegadas,
  amostras de liberação, erros do monitor e contagem de RMWs L1.
- Teste de regressão do extrator: seleciona a primeira ROI entre dois blocos e
  rejeita resultados e chegadas incorretos.
- Matriz de campanha conferida: 270 casos principais, 24 O3 e 15 auto, sem
  duplicatas. Esses números descrevem casos gerados, não execuções concluídas.
- Instrumentação, micro centralizada, P=4, W=0, 20 iterações:
  162.625 ticks/iteração com eventos e 142.450 sem eventos (aproximadamente
  14,2% de acréscimo em relação à execução sem eventos). Uma execução por caso;
  este número não é uma estimativa com intervalo de confiança.

## Bloqueio observado com O3

O binário local `gem5.opt` (v25.1.0.1, compilado em 20/10/2026) terminou com
SIGSEGV em `gem5::o3::Decode::sortInsts` ao testar micro centralizada com quatro
threads. A falha também ocorreu com uma thread, `--no-monitor` e o executável
`micro-noevents`. Isso não identifica a causa do defeito nem comprova equivalência
com o simulador original. É necessário investigar/recompilar o ambiente antes
de validar os casos O3.

## Ainda necessário para afirmar conformidade final

- Resolver o bloqueio O3 e concluir a campanha completa, com três sementes.
- Medir o limiar e avaliar os 15 casos auto; limiar 4 nos testes não é calibrado.
- Comparar as estatísticas de CPU/cache do gem5 modificado sem monitor com as
  do original no mesmo ambiente (critério v da p. 408).
- Medir a linha de base sequencial no simulador para o speedup correspondente;
  a referência nativa atual valida o resultado, não serve como baseline temporal.
- Produzir gráficos com barras de erro e a análise exigida na seção 11.11.
- Arquivar os dados da campanha junto ao relatório. Os testes desta sessão
  ficaram em `/tmp/opencode/barrier-validation-final`, não integram o patch.

A branch main não foi atualizada: os testes de correção concluídos não substituem
essas verificações experimentais e os requisitos da entrega final.
