# gem5 Adaptive Barrier

`barreiras-adaptativas-gem5.patch` contém o projeto completo, incluindo a correção das bordas
do Jacobi. A partir de um clone limpo do gem5 no commit de `versao-base.txt`:

```sh
git checkout 62c7bf284864b83f7308f5e14ca9c80812621c29
git apply --check /caminho/para/barreiras-adaptativas-gem5.patch
git apply /caminho/para/barreiras-adaptativas-gem5.patch
scons -j4 build/X86/gem5.opt
scons -C util/m5 build/x86/out/libm5.a

make -C workloads/p02

build/X86/gem5.opt -d m5out/p02-demo configs/projetos/p02/sistema_p02.py \
    --prog micro --threads 2 --alg central --work 0 --iters 20 --cpu timing
```

## Testes e campanha

Executar a partir da raiz do gem5:

```sh
python3 workloads/p02/test_workloads.py
python3 configs/projetos/p02/test_extrai_p02.py
python3 configs/projetos/p02/test_campanha_p02.py
workloads/p02/barrier_stress

# 45 casos pequenos; limiar 4 serve apenas para exercitar os dois ramos de auto.
python3 configs/projetos/p02/campanha_p02.py build/X86/gem5.opt \
    --stage smoke --iters 4 --limiar 4 --out m5out/validacao

# 270 casos principais e 24 de confirmação O3.
python3 configs/projetos/p02/campanha_p02.py build/X86/gem5.opt \
    --stage main --out m5out/principal
python3 configs/projetos/p02/campanha_p02.py build/X86/gem5.opt \
    --stage o3 --out m5out/o3

# Depois de determinar LIMIAR com os dados da campanha, executar os 15 casos auto.
python3 configs/projetos/p02/campanha_p02.py build/X86/gem5.opt \
    --stage auto --limiar "$LIMIAR" --out m5out/adaptativa
```

Cada execução guarda `run.json`, `stdout.txt`, `stderr.txt` e `stats.txt`.
A referência é executada nativamente, com `--sequential 1`, sem criar threads
nem executar m5ops. Na integração ela preserva a divisão das parcelas para o P
avaliado. O programa integra `4/(1 + a*x*x)` em `[0,1]` por pontos médios,
inicializa `a=1` e, após cada redução, atualiza `a=1+integral/4`.

Ao terminar, a campanha gera `results.csv`. O extrator usa o primeiro bloco
completo da ROI e reprova diferenças de resultado, episódios, chegadas,
amostras de liberação, erros do monitor e RMWs L1 incompatíveis com a barreira.
Também pode ser executado separadamente:

```sh
python3 configs/projetos/p02/extrai_p02.py m5out/principal --csv principal.csv
```

Os RMWs L1 são descontados das duas barreiras de controle dentro da ROI (2P).
Uma variável atômica de publicação impede a segunda barreira de começar antes
do reset. Invalidações, faltas e RMWs L2 incluem o controle da ROI; não são
atribuíveis exclusivamente à barreira medida. `speedup_p1` e `efficiency_p1`
usam o caso de uma thread do mesmo algoritmo, não o tempo da versão sequencial.

Para medir o custo dos eventos mantendo as chamadas de reset/dump:

```sh
make -C workloads/p02 micro-noevents
build/X86/gem5.opt -d m5out/sem-eventos configs/projetos/p02/sistema_p02.py \
    --prog micro-noevents --threads 4 --alg central --work 0 --iters 20
```

Consulte `VALIDACAO.md` para os resultados executados e as pendências da entrega.
