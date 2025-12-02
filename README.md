# Sistema Interativo de Processamento e Zoom de Imagens

## Descrição do Projeto

Este projeto implementa uma aplicação completa em linguagem C para processamento de imagens BMP na placa DE1-SoC, permitindo operações avançadas de zoom e seleção de regiões de interesse através de uma interface interativa baseada em mouse. O sistema utiliza a API desenvolvida em Assembly para comunicação com o coprocessador gráfico na FPGA, proporcionando uma experiência fluida de manipulação de imagens em tempo real.

O principal objetivo desta aplicação é fornecer uma interface de alto nível que aproveite todo o potencial do coprocessador de imagens implementado em Verilog, oferecendo funcionalidades como carregamento de imagens BMP, seleção e centralização de regiões específicas, zoom interativo controlado por mouse e restauração da imagem original. O sistema gerencia toda a comunicação entre o software e o hardware, garantindo sincronização, eficiência e uma experiência de usuário intuitiva.

## Sumário

* [Arquitetura do Sistema](#arquitetura)
* [Estruturas de Dados e Gerenciamento de Memória](#estruturas)
* [Carregamento e Processamento de Imagens BMP](#bmp)
* [Sistema de Seleção de Região](#selecao)
* [Modo de Zoom Interativo com Mouse](#zoom)
* [Interface de Menu e Fluxo de Operações](#interface)
* [Análise dos Resultados e Demonstração](#resultados)
* [Referências](#referencias)

## <a id="arquitetura"></a>Arquitetura do Sistema

A arquitetura da aplicação foi projetada em camadas, separando claramente as responsabilidades de cada componente e garantindo modularidade e manutenibilidade do código. O sistema opera sobre três camadas principais que trabalham de forma integrada para proporcionar uma experiência completa de processamento de imagens.

### Camada de Hardware

A camada de hardware é composta pela FPGA e pela API em Assembly, responsável pela comunicação direta com o coprocessador gráfico. Esta camada gerencia o barramento Lightweight HPS-to-FPGA, controla os registradores de entrada e saída paralela, e executa as operações de processamento de imagem no hardware dedicado. A utilização de componentes de hardware especializados permite que operações intensivas sejam realizadas com alta performance e baixa latência.

### Camada de Interface

A camada de interface, implementada em linguagem C, representa o núcleo lógico da aplicação. Nesta camada são implementadas todas as funcionalidades de alto nível, incluindo manipulação de arquivos BMP, gerenciamento de buffers de imagem, controle de eventos de mouse e coordenação das operações do coprocessador. A separação entre esta camada e o hardware subjacente facilita manutenção e permite expansão futura das funcionalidades.

### Camada de Apresentação

A camada de apresentação fornece uma interface textual intuitiva que guia o usuário através das funcionalidades disponíveis. O menu interativo oferece feedback visual detalhado sobre o progresso das operações, exibindo mensagens informativas, indicadores de progresso e alertas de erro quando necessário. Esta camada foi projetada para maximizar a usabilidade e tornar o sistema acessível mesmo para usuários sem conhecimento técnico profundo.

### Fluxo de Dados

O fluxo de dados no sistema segue um caminho bem definido que garante integridade e consistência das operações. Inicialmente, a entrada do usuário é capturada através do menu interativo, onde as escolhas são validadas e interpretadas. Em seguida, as funções implementadas em C processam a requisição, preparando os dados necessários e formatando-os adequadamente para comunicação com o hardware.

A comunicação com o hardware é realizada através da API em Assembly, que encapsula os detalhes de baixo nível do barramento e dos registradores. As instruções são então enviadas à FPGA, onde os algoritmos implementados em Verilog executam o processamento efetivo da imagem. Finalmente, o status da operação é retornado através das mesmas camadas, sendo eventualmente exibido ao usuário na forma de mensagens e feedback visual.

## <a id="estruturas"></a>Estruturas de Dados e Gerenciamento de Memória

### Estruturas do Formato BMP

O sistema implementa as estruturas padronizadas do formato BMP para garantir leitura e interpretação corretas dos arquivos de imagem. Estas estruturas seguem a especificação oficial da Microsoft e permitem acesso aos metadados essenciais da imagem.

A primeira estrutura, denominada BMPHeader, armazena informações gerais do arquivo. Ela contém o identificador de tipo que deve sempre ser os caracteres "BM", o tamanho total do arquivo em bytes, dois campos reservados que devem ser zero, e o offset que indica onde começam os dados reais dos pixels dentro do arquivo.

A segunda estrutura, BMPInfoHeader, contém informações detalhadas sobre a imagem em si. Esta inclui o tamanho do próprio cabeçalho, as dimensões da imagem em pixels (largura e altura), o número de planos de cor (sempre um para BMP), a quantidade de bits por pixel, o tipo de compressão utilizado, o tamanho da imagem em bytes, as resoluções horizontal e vertical, e informações sobre a paleta de cores quando aplicável.

### Estrutura de Seleção de Região

Para possibilitar a seleção interativa de áreas de interesse na imagem, foi implementada uma estrutura dedicada que mantém todas as informações necessárias sobre a região sendo selecionada. Esta estrutura armazena as coordenadas X e Y do primeiro ponto clicado pelo usuário, bem como as coordenadas do segundo ponto que define o canto oposto do retângulo de seleção.

Além das coordenadas, a estrutura mantém flags de controle que indicam o estado atual da seleção. Uma flag sinaliza se o usuário está atualmente arrastando o mouse durante a seleção, enquanto outra indica se existe uma seleção válida e completa pronta para ser processada. Este mecanismo permite implementar diferentes modos de seleção e validar adequadamente as regiões escolhidas.

### Sistema de Backup de Imagem

O sistema mantém um buffer global que armazena a imagem original completa, permitindo operações de restauração e múltiplas transformações sem perda de qualidade acumulada. Este buffer é alocado dinamicamente durante o carregamento da primeira imagem e é mantido em memória durante toda a execução da aplicação.

O buffer armazena exatamente 76.800 pixels correspondentes à resolução padrão de 320x240 pixels, onde cada pixel é representado por um único byte em escala de cinza. Esta abordagem garante uso eficiente de memória ao mesmo tempo que permite acesso rápido aos dados originais sempre que necessário para operações de restauração ou processamento adicional.

## <a id="bmp"></a>Carregamento e Processamento de Imagens BMP

### Processo de Carregamento

O carregamento de imagens BMP é realizado através de um pipeline completo que garante validação adequada e conversão correta dos dados. Este processo foi projetado para ser robusto e informativo, fornecendo feedback detalhado ao usuário durante toda a operação.

### Abertura e Validação

O primeiro passo consiste na abertura do arquivo especificado e leitura dos cabeçalhos. O sistema lê primeiramente o BMPHeader para obter informações gerais do arquivo, seguido pela leitura do BMPInfoHeader que contém os detalhes da imagem. Uma validação rigorosa é então realizada para garantir que o arquivo é realmente um BMP válido, verificando o identificador de tipo e confirmando que as dimensões e formato de pixel são suportados pelo sistema.

Durante a validação, o sistema verifica se a imagem possui as dimensões esperadas de 320x240 pixels e se está em um dos formatos suportados: escala de cinza de 8 bits ou RGB de 24 bits. Qualquer desvio destas especificações resulta em uma mensagem de erro clara e interrupção do processo de carregamento.

### Preparação e Conversão

Após validação bem-sucedida, o sistema prepara-se para a leitura dos dados de pixel. O cálculo do tamanho de cada linha considera o padding necessário, uma vez que no formato BMP cada linha deve ter um número de bytes múltiplo de quatro. Buffers temporários são alocados para facilitar o processamento eficiente dos dados.

A leitura da imagem é realizada linha por linha, considerando que o formato BMP armazena as linhas de baixo para cima. Para cada pixel lido, o sistema determina se é necessária conversão de formato. Pixels já em escala de cinza de 8 bits são copiados diretamente, enquanto pixels RGB de 24 bits são convertidos para escala de cinza através do cálculo da média aritmética dos componentes vermelho, verde e azul.

### Armazenamento e Transmissão

Cada pixel processado é simultaneamente armazenado no buffer de backup global e enviado para a memória da FPGA através da função de escrita de pixel da API. Este processo dual garante que uma cópia da imagem original esteja sempre disponível para operações futuras, enquanto a FPGA recebe imediatamente os dados para processamento.

Durante todo o processo de envio, o sistema monitora e exibe o progresso em tempo real. A cada 500 pixels processados, uma atualização é exibida mostrando a quantidade de pixels já enviados, o total esperado e a porcentagem de conclusão. Este feedback constante permite ao usuário acompanhar a operação e estimar o tempo restante.

### Tratamento de Formatos

O sistema suporta dois formatos principais de imagem BMP. Imagens em escala de cinza com 8 bits por pixel não necessitam conversão e são processadas diretamente, pixel por pixel. Já imagens RGB com 24 bits por pixel passam por um processo de conversão onde cada pixel é transformado em escala de cinza calculando-se a média aritmética de suas componentes de cor: vermelho, verde e azul. Esta abordagem simples mas eficaz produz resultados visualmente satisfatórios para a maioria das imagens.

## <a id="selecao"></a>Sistema de Seleção de Região

### Seleção Interativa

O sistema de seleção de região implementa uma interface intuitiva baseada em dois cliques do mouse, permitindo ao usuário escolher facilmente uma área retangular de interesse na imagem. Este mecanismo é essencial para operações de zoom focado e análise de detalhes específicos.

### Modo de Seleção Ativo

Quando o usuário ativa o modo de seleção através do menu, o sistema entra em um estado especial onde captura e processa eventos do mouse em tempo real. Uma interface guiada é exibida com instruções claras sobre como proceder. O cursor do mouse é rastreado continuamente e suas coordenadas são atualizadas em tempo real na tela VGA, proporcionando feedback visual imediato ao usuário.

Durante este modo, o usuário pode ver exatamente onde está posicionando o cursor, facilitando a seleção precisa da região desejada. O sistema permanece neste estado aguardando as ações do usuário, que podem ser a definição dos pontos de seleção ou o cancelamento da operação.

### Definição de Pontos

O primeiro clique do botão esquerdo do mouse marca o primeiro canto do retângulo de seleção. As coordenadas deste ponto são capturadas e armazenadas na estrutura de seleção. Uma mensagem confirma o registro do primeiro ponto e instrui o usuário a clicar novamente para definir o canto oposto.

O segundo clique do botão esquerdo completa a definição da região. As coordenadas deste segundo ponto são armazenadas e o sistema imediatamente aciona o processamento da região selecionada. Não importa a ordem em que os cantos são clicados, pois o sistema automaticamente normaliza as coordenadas para garantir que formem um retângulo válido.

A qualquer momento durante o processo de seleção, o usuário pode pressionar o botão direito do mouse para cancelar a operação. Isso interrompe o modo de seleção e retorna ao menu principal sem aplicar quaisquer mudanças à imagem.

### Processamento da Região

Uma vez que a região tenha sido completamente definida pelos dois cliques, o sistema inicia o processamento para centralizar a área escolhida e aplicar máscara ao restante da imagem. Este processo envolve várias etapas de validação e transformação de coordenadas.

### Normalização e Validação

Primeiramente, o sistema normaliza as coordenadas recebidas, garantindo que os valores mínimos e máximos estejam corretamente ordenados independentemente da ordem em que o usuário clicou. As coordenadas são então mapeadas do espaço da tela VGA de 640x480 pixels para o espaço da imagem de 320x240 pixels, considerando o offset de centralização inicial.

Uma série de validações é então realizada para garantir que a seleção é válida. O sistema verifica se a região está completamente dentro dos limites da imagem, se possui dimensões mínimas adequadas para processamento útil, e se não há problemas com as coordenadas que possam causar erros durante o processamento.

### Cálculo de Centralização

Com a região validada, o sistema calcula os parâmetros necessários para centralizá-la na tela. A largura e altura da região são determinadas pela diferença entre as coordenadas máximas e mínimas. Os offsets de centralização são então calculados para posicionar a região selecionada exatamente no centro da área de visualização de 320x240 pixels.

Este cálculo matemático garante que a região de interesse fique perfeitamente centralizada, independentemente de seu tamanho ou posição original na imagem. O restante da área de visualização será preenchido com pixels pretos, criando uma máscara que destaca claramente a região selecionada.

### Renderização Final

O processo de renderização percorre todos os 76.800 pixels da imagem de destino. Para cada posição, o sistema determina se aquele pixel pertence à região selecionada centralizada ou se está fora dela. Pixels dentro da região são copiados do buffer de backup original, preservando exatamente a informação original daquela área. Pixels fora da região recebem o valor zero, correspondente à cor preta, criando o efeito de máscara.

Cada pixel processado é imediatamente enviado para a FPGA através da API, garantindo que a imagem visualizada reflita precisamente a operação de seleção e centralização. O sistema exibe mensagens de progresso durante este processo, informando o usuário sobre o andamento da operação.

## <a id="zoom"></a>Modo de Zoom Interativo com Mouse

### Controle Dinâmico de Zoom

O modo de zoom interativo representa uma das funcionalidades mais sofisticadas do sistema, permitindo que o usuário explore a imagem através de controles naturais e intuitivos do mouse. Este modo combina rastreamento de movimento, detecção de eventos de scroll e comunicação em tempo real com o coprocessador de hardware.

### Eventos de Mouse

O sistema monitora continuamente os eventos gerados pelo mouse, processando movimentos, cliques e ações de scroll wheel. Cada tipo de evento dispara comportamentos específicos que proporcionam uma experiência de uso fluida e responsiva.

Quando o usuário rola a scroll wheel do mouse para cima, o sistema interpreta isto como uma requisição de zoom in, ou seja, ampliar a imagem. A cada evento de scroll up, um comando de zoom in é enviado ao coprocessador, que executa o algoritmo de ampliação correspondente. O sistema alterna entre dois algoritmos diferentes a cada zoom: vizinho mais próximo e replicação de pixels, proporcionando resultados visuais variados.

Quando o usuário rola a scroll wheel para baixo, o sistema interpreta isto como zoom out, ou seja, reduzir a imagem. Similarmente ao zoom in, o sistema alterna entre dois algoritmos: média de blocos e decimação. Esta alternância automática permite ao usuário experimentar diferentes técnicas de processamento sem necessidade de seleção manual.

### Rastreamento de Posição

O movimento do cursor do mouse é rastreado continuamente durante o modo zoom. O sistema mantém coordenadas acumuladas que representam a posição atual do cursor na tela de 640x480 pixels. Estas coordenadas são atualizadas dinamicamente conforme o usuário move o mouse, permitindo que operações de zoom sejam centradas na posição atual do cursor.

O sistema implementa mecanismos de clipping para garantir que as coordenadas permaneçam sempre dentro dos limites válidos da tela. Quando o cursor alcança as bordas, os valores são limitados apropriadamente, evitando que coordenadas inválidas sejam enviadas ao hardware.

As coordenadas atualizadas são periodicamente enviadas ao coprocessador através da função de envio de coordenadas da API. Isto permite que o hardware saiba onde está o foco de interesse do usuário, possibilitando futuras implementações de zoom centrado no cursor.

### Algoritmos de Zoom

O sistema utiliza quatro algoritmos diferentes de zoom, alternando automaticamente entre eles para proporcionar variedade visual e permitir comparação de técnicas.

Para zoom in, o algoritmo de vizinho mais próximo replica cada pixel para múltiplas posições, criando um efeito de ampliação blocado mas rápido. O algoritmo de replicação oferece uma ampliação ligeiramente mais suave através de técnicas de interpolação básica.

Para zoom out, o algoritmo de média calcula a média de blocos de pixels adjacentes, criando uma redução suave e sem aliasing. O algoritmo de decimação simplesmente descarta pixels alternados, produzindo uma redução mais rápida mas potencialmente com perda de detalhes.

### Feedback Visual

Durante todo o tempo que o modo zoom está ativo, o sistema exibe informações em tempo real sobre as operações sendo realizadas. Cada aplicação de zoom é acompanhada de uma mensagem indicando qual algoritmo está sendo utilizado e qual a posição atual do cursor. Este feedback constante ajuda o usuário a entender o comportamento do sistema e acompanhar as transformações sendo aplicadas à imagem.

### Saída do Modo

O usuário pode sair do modo zoom a qualquer momento pressionando o botão esquerdo do mouse. Esta ação interrompe o loop de processamento de eventos e retorna o sistema ao menu principal, mantendo a última versão processada da imagem visível na tela.

## <a id="interface"></a>Interface de Menu e Fluxo de Operações

### Menu Principal

A interface do sistema é baseada em um menu textual estruturado que apresenta todas as funcionalidades disponíveis de forma clara e organizada. O menu é exibido em um formato visualmente destacado com bordas e separadores que facilitam a leitura e navegação.

As opções são numeradas sequencialmente de um a cinco, permitindo que o usuário faça sua seleção simplesmente digitando o número correspondente. Cada opção possui uma descrição concisa mas informativa que explica claramente sua função, eliminando ambiguidades e facilitando a escolha correta.

### Inicialização do Sistema

Antes de apresentar o menu ao usuário, o sistema executa uma sequência de inicialização que prepara todos os componentes necessários para operação. Esta sequência inclui a inicialização da biblioteca de comunicação com a FPGA, abertura do dispositivo de entrada do mouse e configuração inicial do coprocessador.

Durante a inicialização, o sistema valida cada etapa e exibe mensagens informativas sobre o progresso. Se algum erro ocorrer durante esta fase, mensagens de erro claras são exibidas e o sistema é encerrado de forma segura, evitando estados inconsistentes ou comportamentos inesperados.

A inicialização bem-sucedida é confirmada através de mensagens positivas que tranquilizam o usuário de que todos os componentes estão funcionando corretamente e o sistema está pronto para uso.

### Fluxo de Carregamento de Imagem

Quando o usuário seleciona a opção de carregar uma imagem BMP, o sistema inicia o processo de leitura e validação do arquivo especificado. O nome do arquivo pode ser fornecido de forma estática ou solicitado ao usuário, dependendo da configuração da aplicação.

O sistema então valida o formato do arquivo, verifica suas dimensões e tipo de pixel, e inicia o processo de conversão e envio dos dados para a FPGA. Durante toda esta operação, mensagens informativas são exibidas periodicamente, mantendo o usuário informado sobre o progresso.

Ao final do carregamento bem-sucedido, uma cópia completa da imagem é armazenada no buffer de backup e o coprocessado
