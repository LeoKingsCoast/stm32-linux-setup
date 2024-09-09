# Setup STM32 sem STM32CubeIDE

## Introdução

Esse repositório é um registro pessoal para preparar um sistema de desenvolvimento de projetos com microcontroladores ST sem a necessidade da utilização da STM32CubeIDE, que deixo disponibilizado na esperança de ajudar quem deseje ou necessite de um setup parecido. Esse setup utiliza a STM32CubeMX para geração do código de preparação inicial, sendo possível utilizar um software separado para fazer o upload do código para o microcontrolador utilizando um dispositivo ST-Link ou um de seus clones, além de permitir a edição do código com uma IDE ou editor de texto de sua escolha.

**Nota:** Esse guia assume que você está usando uma distribuição Linux baseada em Debian ou Arch, devem haver pacotes equivalentes aos utilizados aqui para Fedora e derivados. Acredito que também deve funcionar em MacOS.

## Programas necessários

### Instalação Mínima

É necessário obter o pacote `stlink-tools` para fazer o upload dos códigos para
a placa, o `gcc-arm-none-eabi` para fazer a compilação com o gcc.

- Instalando com apt:
```bash
sudo apt install stlink-tools gcc-arm-none-eabi
```

- Instalando com pacman:
```bash
sudo pacman -S stlink arm-none-eabi-gcc arm-none-eabi-newlib
```

Esses pacotes são suficientes para escrever um código funcional e fazer o upload para uma placa STM32. Se quiser utilizar a CubeMX para gerar o código de setup e usar o framework HAL, baixe o programa pelo site da ST, descompacte a pasta de instalação para o Linux e siga as instruções do Readme deles. Ao final de uma instalação padrão, o executável da STM32CubeMX estará na seguinte localização:
`/usr/local/STMicroelectronics/STM32Cube/STM32CubeMX/STM32CubeMX`.

### Ferramentas para debug

Para fazer o debug do código durante execução na placa é necessário instalar o pacote `openocd` para iniciar um servidor de debug. Além disso, instalar ou `gdb-multiarch` ou `arm-none-eabi-gdb`, que irá ler e interpretar o código executável, e permitir funções como setar breakpoints e executar o código instrução por instrução. 

- Instalando com apt:
```bash
sudo apt install openocd gdb-multiarch
```

- Instalando com pacman:
```bash
sudo pacman -S openocd arm-none-eabi-gdb
```

### Autopreenchimento de código fora da CubeIDE

Se você estiver usando um editor de texto diferente com LSP de C, ele provavelmente não saberá ler a estrutura de diretórios criada pela CubeMX para seu projeto, e não irá identificar as funções e definições do HAL para configurar autopreenchimento de código, indicações de erro, entre outros. Se você desejar essas funcionalidades, pode usar o [bear](https://github.com/rizsotto/Bear) para gerar um arquivo JSON que seu LSP possa ler ao compilar o projeto.
```bash
sudo apt install bear
sudo pacman -S bear
# Executar no diretório do projeto
bear -- make # O comando make compila seu projeto, o bear lê cada comando executado e gera o JSON para seu LSP
```

## Configurações iniciais

- Conecte o programador st-link ao seu computador ligado à placa STM32. Execute o seguinte comando para verificar se seu computador detectou o dispositivo.
```bash
st-info --probe # Deve retornar "Found x stlink programmers" e outras informações
```

## Gerando o código com a CubeMX

Para gerar o código com a CubeMX, realize o processo normal de configuração do seu microcontrolador de escolha. O ponto de atenção é na aba "Project Manager".

Vá para "Project Manager". Após escolher o nome e localização do seu projeto, configure "Toolchain / IDE" como "Makefile". Após isso, vá para "Code Generator" e selecione "Copy only the necessary library files", e deixe ativa a opção "Generate peripheral initialization as a pair of '.c/.h' files per peripheral"

## Carregando um código no STM32

Após gerar o código e modificá-lo da maneira que desejar, ele deve ser compilado e carregado no STM32. Vá para o diretório do projeto e compile o código.
```bash
cd /path/to/project
make
```

Vá para o diretório `build`, criado durante a compilação. Se você ainda não tiver conectado a placa ao seu computador pelo ST-Link, faça-o agora. A seguir, carregue o arquivo binário do projeto para a placa.
```bash
cd build
st-flash --reset write *project_file.bin* 0x8000000 
```

Pode ser necessário apertar o botão de reset da placa ou desconectá-la do computador e ligar novamente para que o programa seja inicializado.

## Adicionando bibliotecas externas

Para adicionar suas próprias bibliotecas ao projeto gerado pela CubeMX, siga os seguintes passos.

A partir do diretório do seu projeto, adicione seus arquivos fonte em ./Core/Src e os seus headers em ./Core/Inc. Para que o seu arquivo fonte seja compilado, adicione-o à variável C_SOURCES do Makefile gerado pela CubeMX.
```make
C_SOURCES = \
...
Core/Src/your_custom_library.c \ # <-- Adicionar esta linha
...
```

## Debugging

>[!WARNING]
> Este setup funcionava há alguns meses atrás. Atualmente a placa para a execução dentro da função `HAL_Init` ao tentar debugar o código. Irei continuar a procura por uma solução em outro momento.

Para fazer o debug do código, é possível utilizar o gdb juntamente com o OpenOCD.

Abra um servidor com o `openocd`.
```bash
openocd -f /usr/share/openocd/scripts/interface/stlink.cfg -f /usr/share/openocd/scripts/target/stm32f1x.cfg
```

Vá até o diretório `build` do seu projeto e abra o  arquivo `.elf` com o
debugger `gdb-multiarch`. A seguir, conecte o debugger ao servidor criado anteriormente.
```bash
gdb-multiarch ./nome_do_projeto.elf
target extended-remote localhost:3333 # Executar na interface do gdb
```

Ou, se estiver no Arch:
```bash
arm-none-eabi-gdb ./nome_do_projeto.elf
target extended-remote localhost:3333 # Executar na interface do gdb
```

Para usar o debugger, você pode usar o comando `b main` para setar um breakpoint na função main, `run` para rodar o projeto do início, e `next` para avançar para as próximas instruções. Ler a documentação do `gdb` para mais detalhes sobre os comandos de debug. Apertar enter sem nenhum prompt irá reexecutar o último comando. O comando `lay next` pode ser usado para obter uma interface com mais informações.

