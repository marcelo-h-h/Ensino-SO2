###  Quando é feito o fork, o SO copia as áreas, copia a pilha, o que há no topo da pilha?
No topo da pilha, está salvo o contexto de execução.

###  Qual a diferença entre o BSS e o HEAP
O bss é uma area marcada no programa executavel mas que nao está no programa executável, em que o compilador quando montando o programa, cria essa área estática. O linux mapeia o BSS para páginas preenchidas com 0.
O HEAP também não existe no programa executável, é a área de memória usada para alocações dinâmicas. É gerenciado pelas bibliotecas do maloc e free.  É possível aumentar ou diminuir essa área de HEAP com brk() e sbrk().

Rodou /src/mem/end em background e fez more /proc/numeroprocessoend/maps

###  O que é um memory leak?
O programa vai alocando memória, alocando memória sem liberar, ocorre uma lotação da memória, sendo necessário até mesmo reiniciar o computador.

### O que é uma variável e qual a influência do tipo?
Uma variável é um endereço de memória, seu tipo influência o tamanho da área de memória alocado pra ela.

more /src/mem/pagesize.c
Espera-se como retorno o tamanho da página de memórai

### O que é um arquivo?
DO ponto de vista do processo um arquivo é uma sequencia de bytes pra qual também é mantido pelo SO uma posição (offset) corrente.
Do ponto de vista do SO um arquivo são metadados + um conjunto de blocos de disco.
Os blocos são tipicamente de 4k pois corresponde ao tamanho da página de memória e por isso tende a ser mais eficiente

### Como são identificados blocos no disco?
O SO enxerga o disco como uma geometria, em que os blocos são identificados pela posição no cilindro.

### Como é feita uma operação de leitura?
Antes de ler o arquivo, é necessário fazer um
```C
int fd = open(path, flags, mode);
.
.
.
n = read(fd, buf, count);
.
.
.
n = write(fd, buf, count);
.
.
.
close(fd);
```

O path pode ser relativo ou absoluto se iniciado com '/'.
O buf na leitura é o endereço da posição de memória em que o programa quer que o SO coloque os dados.
Quando o read retorna, os dados vão estar disponiveis no buf, pois é uma operação bloqueante, ao menos que se utilize flags para tornar não bloqueante, daí então pode retornar sem que os dados estejam no buf.


### De que maneira a oportunidade de leitura de arquivos pode ser uma oportunidade de troca de contexto?
Os controladores de disco, com os quais o Device Software interage
Quando o programa precisa de uma operação de entrada e saida o SO deve determinar com base no OFFSET e no numero de blcoos envolvidos, o SO vai ler o conjunto de blocos do disco e conjunto de blocos de memoria e ver se ja ha blocos do disco na memoria. Quando o programa verifica se ja estao na memoria, se não estao vai reservar bloco e instruir o controlador a transferir os blocos para a memoria.
O SO não fica esperando, ele salva o estado e enquanto espera executa instruções de outro processo
As chamadas de sistema que envolvem leitura de arquivo normalmente possuem no linux 2 partes, tophealth e bottom health,  a primeira vai até a instrução de compilador e a segunda parte é a partir da interrupção do controlador que diz que a instrução já foi feita.

### Para o "aio_read()" ou operação de leitura assíncrona, o que pode ser um evento ou notificação para o processo quando a operação estiver pronta?
Tipicamente um sinal, por exemplo, um SIGIO

###  O que é mmap()?
É uma chamada do SO para mapear um arquivo ou parte dele na memória.
Supondo-se um arquivo lógico e um espaço de endereçamento.

______

###  O que é pipe?
No pipe, ele utiliza as duas próximas posições no vetor de arquivos abertos do processo para o pipe


###  O que acontece quando fazemos p1 | p2 ?
O shell vai interpretar, criar um pipe, fazer fork para criar p1, dup2 no pipefd[1]  para a posição 1 do vetor de arquivos abertos, forka e cria p2 e realiza também a mudança no pipefd, faz exec de p1 e p2 e esperak.

###  O que é um fifo?
Fifo é um tipo de pipe nomeado, porém com apenas uma entrada no vetor de arquivos aberta conforme determinado na execução. Serve para que processos distintos consigam referenciar o mesmo pipe.

______

###  Como compilar um programa?
gcc programa.c -o programa
Pode usar -Wall para ver os warnings

###  O que é um makefile?
Quando utiliza o make, ele busca no makefile pelo nome da regra e executa os comando, seguindo as dependências da regra.
Quando não há makefile, o make tem regras padrões para tipos diferentes de programa

###  Como é feita uma chamada de sistema? O que é uma chamada de sistema?
Uma chamada de sistema é um serviço do SO. São usadas através de linguagens de alto nível, que possuem bibliotecas de funções que possuem interfaces para chamadas de sistema(Wrappers). A dificuldade é localizar o código do SO em tempo de execução e mudar do anel de privilégio 3 para o anel de privilégio 0, que é necessário para execução de chamadas.

###  Como a libc faz uma chamada de sistema?
Via interrupção ou via syscall:
int: Acessa a memória para buscar o vetor de interrupção
syscall: Já tem o endereço dentro da CPU

###  Como termina um programa?
Exit()


###  Como é criado um processo?
Através de uma instrução fork(). Vale lembrar os procedimentos do SO. Cópias e tudo mais.


###  Como alterar a imagem de um processo existente?
exec(), vai alternar a imagem do processo corrente com o processo que foi passado à instrução.


###  Como um processo espera por um processo filho?
wait(), wait4(), waitpid()


###  Como é tratado um sinal?
Se não tratado, o padrão é o definido pelo posix.
Capturar um sinal significa esperar por um sinal e se recebido executar uma rotina.
Os sinais SIGKILL e SIGSTOP não podem ser ignorados.
Bloquear um sinal é deixá-lo pendente quando recebido.


###  O que se usa para controle de tempo?(Setar um timer por exemplo)
Utiliza funções de setitimer por exemplo que enviam sinal quando terminam.


###  O que acontece quando tenta-se dividir por 0?
O Processador gera uma excessão ao tentar executar a instrução e gera uma interrupção, desviando o endereço para um código do SO que gera um sinal, que por default, termina o processo.


###  Como o shell limita o tempo de cpu dos processos criados?
set resource limit - mecanismos do SO que limitam o tempo máximo.
o shell ajusta esses parametros e aplica para cada processo filho que ele cria.
```ulimit -a```



###  Qual a politica padrão de escalonamento do linux?
A politica padrão é chamado de completely fair schedule - que procura garantir que todas as fraçoes de uso estejam na médi.
Como o shell sabe o tempo de vida dos processo, ele consegue calcular a fração de uso.
 
