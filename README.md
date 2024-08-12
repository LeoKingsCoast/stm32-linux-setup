# Setup STM32 sem STM32CubeIDE

## Introdução

Esse repositório é um registro pessoal para preparar um sistema de desenvolvimento de projetos com microcontroladores ST sem a necessidade da utilização da STM32CubeIDE. Esse setup utiliza a STM32CubeMX para geração do código de preparação inicial, mas com ele, é possível utilizar um software separado para fazer o upload do código para o microcontrolador utilizando um dispositivo ST-Link ou um de seus clones, além de permitir a edição do código com uma IDE ou editor de texto de sua escolha. Disponibilizo este guia publicamente para aqueles que prefiram ou necessitem de um sistema independente da IDE.

**Nota:** Esse guia assume que você está usando uma distribuição Linux. Acredito que também deve funcionar em MacOS 

## Programas necessários

## Configurações iniciais

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

Para fazer o debug do código, é possível utilizar o gdb juntamente com o OpenOCD.
