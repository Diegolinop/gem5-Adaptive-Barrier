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
  duplicatas. A geração da matriz não significa que a campanha de desempenho
  completa foi executada.
- Após o backport do TimeBuffer e a recompilação, Hello world passou tanto em
  TimingSimpleCPU quanto no O3, sem caches nem instrumentação do projeto.
- Os 24 casos O3 passaram com quatro iterações por execução: micro W=1000 e
  Jacobi N=512, centralizada/disseminação, P=4,16, sementes 1,2,3. Resultados,
  episódios, chegadas, amostras de liberação, erros do monitor e RMWs L1 foram
  verificados pelo extrator. São testes funcionais, não a campanha de 100
  iterações usada por padrão no script.
- Instrumentação, micro centralizada, P=4, W=0, 20 iterações:
  162.625 ticks/iteração com eventos e 142.450 sem eventos (aproximadamente
  14,2% de acréscimo em relação à execução sem eventos). Uma execução por caso;
  este número não é uma estimativa com intervalo de confiança.

## Correção do bloqueio observado com O3

O binário local `gem5.opt` (v25.1.0.1, compilado em 20/10/2026) terminou com
SIGSEGV em `gem5::o3::Decode::sortInsts` ao testar micro centralizada com quatro
threads. A falha também ocorreu com uma thread, `--no-monitor` e o executável
`micro-noevents`. O Hello world original também reproduziu a falha sem caches.

Foi aplicado em `src/cpu/timebuf.hh` o mesmo diff do commit oficial
`cb8b6674828a290fcd6f04c900b517e421fbe7ed` ("cpu: Use value-initialization for
TimeBuffer entries"): substitui memset seguido de placement-new com inicialização
padrão por placement-new com inicialização por valor, no construtor e na
reciclagem do buffer. Isso faz a inicialização seguir as regras da linguagem,
em vez de depender dos bytes zerados antes da construção do objeto.

O novo gem5 foi compilado com GCC 16.2.1 e Python 3.12.16; o SCons informa que
GCC 16 não é oficialmente suportado por essa versão. Após a correção e a
recompilação, o SIGSEGV não reapareceu nos testes listados acima. Não foi feita
uma segunda recompilação sem a correção no ambiente atualizado; portanto, os
testes demonstram o funcionamento do estado corrigido, sem isolar todos os
efeitos da mudança do ambiente de compilação.

Dados e CSV dos testes O3: `/tmp/opencode/o3-timebuf-fixed`.

## Ainda necessário para afirmar conformidade final

- Concluir a campanha de desempenho completa, com três sementes.
- Medir o limiar e avaliar os 15 casos auto; limiar 4 nos testes não é calibrado.
- Comparar as estatísticas de CPU/cache do gem5 modificado sem monitor com as
  do original no mesmo ambiente (critério v da p. 408). Para isolar a
  instrumentação, aplicar também o backport do TimeBuffer à linha de base.
- Medir a linha de base sequencial no simulador para o speedup correspondente;
  a referência nativa atual valida o resultado, não serve como baseline temporal.
- Produzir gráficos com barras de erro e a análise exigida na seção 11.11.
- Arquivar os dados da campanha junto ao relatório. Os testes desta sessão
  ficaram em `/tmp/opencode/barrier-validation-final`, não integram o patch.

A branch main não foi atualizada: os testes de correção concluídos não substituem
essas verificações experimentais e os requisitos da entrega final.
