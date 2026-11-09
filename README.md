# gem5 Adaptive Barrier

`barreiras-adaptativas-gem5.patch` contém o projeto completo, incluindo a correção das bordas
do Jacobi. A partir de um clone limpo do gem5 no commit de `versao-base.txt`:

```sh
git checkout 62c7bf284864b83f7308f5e14ca9c80812621c29
git apply --check /caminho/para/barreiras-adaptativas-gem5.patch
git apply /caminho/para/barreiras-adaptativas-gem5.patch
scons -j4 build/X86/gem5.opt
scons -C util/m5 build/x86/out/libm5.a

for program in micro jacobi integ; do
    g++ -O2 -std=c++17 -pthread -static -DUSE_M5OPS -Iinclude \
        "workloads/p02/${program}.cc" workloads/p02/{barreira,central,dissem}.cc \
        util/m5/build/x86/out/libm5.a -o "workloads/p02/${program}"
done

build/X86/gem5.opt -d m5out/p02-demo configs/projetos/p02/sistema_p02.py \
    --prog micro --threads 2 --alg central --work 0 --iters 20 --cpu timing
```

Para a campanha completa e a extração (executar a partir da raiz do gem5):

```sh
python configs/projetos/p02/campanha_p02.py build/X86/gem5.opt --dry-run
python configs/projetos/p02/campanha_p02.py build/X86/gem5.opt
python configs/projetos/p02/extrai_p02.py m5out/p02 --csv p02.csv
```

O extrator falha se o monitor registrar saída antecipada ou evento inválido.
