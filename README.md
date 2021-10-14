# projectCadastroJogos
Projeto cadastro de Jogos C++
#include <stdio.h>
#include <iostream>
#include <stdlib.h>
#include <string.h>
#include <algorithm> 

#define MAXCHAR 100
#define MAXARRAY 100

using namespace std;

typedef struct data {
    int dia;
    int mes;
    int ano;
} data_t;

typedef struct jogos {
    int codigo;
    char titulo[MAXCHAR];
    char produtora[MAXCHAR];
    float valor;
    data_t data_lancamento;
    char plataforma[MAXCHAR];
} jogos_t;

typedef struct array {
    jogos_t *valores; 
    int capacidade;
    int tamanho;
} array_t;

void menu(void){
    for (int i = 0; i < 40; i++)
        cout << '-';
        
    cout << "\n===================\n";
    cout << "| a: adicionar    |\n";
    cout << "| b: buscar	  |\n";
    cout << "| l: lista        |\n";
    cout << "| r: remover      |\n";
    cout << "| f: finalizar    |\n";
    cout << "===================\n";

    for (int i = 0; i < 40; i++)
        cout << '-';
    cout << endl;
}

// inicializa array com valores apropriados
int inicializa_array(array_t *a, int capac, int tam) {
    a->capacidade = capac;
    a->tamanho = tam;
    a->valores = (jogos_t*) malloc(capac*sizeof(jogos_t));
    if (a->valores == NULL){
    	cout << "Erro ao alocar memoria \n";
    	return -2;
	}
}



// adiciona um novo item (jogo)
bool adiciona_jogos(array_t *a) {
    if (a->tamanho == a->capacidade)
        return false;
    int novo_idx = a->tamanho;
    jogos_t *novo_jogos = &(a->valores[novo_idx]);
    cout << "Codigo: ";
    scanf("%d", &novo_jogos->codigo); // cin >> novo_jogos->codigo
    cout << "Titulo: ";
    scanf(" %[^\n]s", novo_jogos->titulo); // cin >> novo_jogos->titulo (???)
    cout << "Produtora: ";
    scanf(" %[^\n]s", novo_jogos->produtora);
    cout << "Data de Lancamento (<dd> <mm> <aaaa>): ";
    scanf("%d %d %d", &novo_jogos->data_lancamento.dia,
                        &novo_jogos->data_lancamento.mes,
                        &novo_jogos->data_lancamento.ano);
    cout << "Valor (R$): ";              
    scanf("%f", &novo_jogos->valor);
    cout << "Plataforma (console ou pc): ";
    scanf(" %[^\n]s", novo_jogos-> plataforma); // cin >> novo_jogos->plataforma
	a->tamanho++;
    return true;
}
 
// exibe informacoes de um item especifico
void mostra_info(jogos_t *item) {
    cout << "Titulo: " << item->titulo << endl;
    cout << "Produtora: " << item->produtora << endl;
    cout << "Codigo: " << item->codigo << endl;
    printf("Lancamento: %02d/%02d/%d\n", item->data_lancamento.dia,
                                            item->data_lancamento.mes,
                                            item->data_lancamento.ano);
    printf("Valor: R$ %.2f\n", item->valor);
    cout << "Plataforma: " << item->plataforma << endl;

}
 
 //ordena��o
 
bool ordernarPorCodigo(jogos_t const& jogo, jogos_t const& jogo_compara){
    return jogo.codigo < jogo_compara.codigo;
}
// lista informacoes de todo o estoque
void lista_estoque(array_t *itens) {
    if (itens->tamanho == 0) {
        cout << "Nao ha itens registrados.\n";
        return;
    }
    jogos_t *item;
    

    sort(itens->valores, itens->valores + itens->tamanho, ordernarPorCodigo);

    for (int i = 0; i < itens->tamanho; i++) {
        item = &(itens->valores[i]);
        mostra_info(item);
        cout << endl;
    }
    return;
}



/* 
Busca um elemento onde o valor de c�digo � igual a "chave"
Retorna um ponteiro para o item encontrado ou NULL, caso n�o encontre
*/
jogos_t* busca_por_codigo(array_t *estoque, int chave) {
    if (estoque->tamanho == 0)
        return NULL;
    for (int i = 0; i < estoque->tamanho; i++) {
        if (estoque->valores[i].codigo == chave)
            return &(estoque->valores[i]);
    }
    return NULL;
}

/* 
Busca um elemento cujo valor de c�digo n�o e igual a "chave"
Retorna um inteiro representando a posi��o do elemento dentro
do vetor, ou -1 caso n�o encontre.
*/
int busca_por_codigo_idx(array_t *estoque, int chave) {
    if (estoque->tamanho == 0)
        return -1;
    for (int i = 0; i < estoque->tamanho; i++) {
        if (estoque->valores[i].codigo == chave)
            return i;
    }
    return -1;
}

/* Remove um elemento do array (vetor) baseado no valor
de "codigo".
Retorna true em caso de sucesso, ou false caso contr�rio.
*/
bool remove_jogos_por_codigo(array_t *estoque, int codigo) {
    int idx;
    if (!estoque || estoque->tamanho == 0)
        return false;
    idx = busca_por_codigo_idx(estoque, codigo);
    if (idx < 0)
        return false;
    for (int i = idx + 1; i < estoque->tamanho; i++) {
        estoque->valores[i-1] = estoque->valores[i];
    }
    estoque->tamanho--;
    return true;
}

//processo de escrever um certo item em arquivo
int escreve_jogo_arquivo(FILE *fp, jogos_t *jogos) {
	if (fp == NULL || jogos == NULL)
	return -1;
	fprintf(fp,"%d\n%.2f\n%d %d %d\n", 
		  jogos->codigo,
		  jogos->valor,
		  jogos->data_lancamento.dia,
		  jogos->data_lancamento.mes,
		  jogos->data_lancamento.ano);
    fprintf (fp, "%s\n%s\n%s\n", jogos->titulo, jogos->produtora, jogos->plataforma);
    return 0;
}
//le um item do arquivo
int le_jogo_arquivo(FILE *fp, jogos_t *jogos){
	if (fp == NULL || jogos == NULL)
	return -1;
    fscanf(fp,"\n%d\n%f\n%d %d %d\n%[^\n\t]\n%[^\n\t]\n%[^\n\t]",   
          &(jogos->codigo),
		  &(jogos->valor),
		  &(jogos->data_lancamento.dia),
		  &(jogos->data_lancamento.mes),
		  &(jogos->data_lancamento.ano),
          &(jogos->titulo),
          &(jogos->produtora),
          &(jogos->plataforma));
		 

    return 0;
}



int main(){
    array_t estoque;
    int chave;
    jogos_t *jogos_res;
    char op;
    FILE *fp;
    int i;
    int tam, capac;
        // verifica se existe arquivo para leitura, carregando os dados se necess�rio
    fp = fopen("ArquivoDoEstoque.txt", "r");

    if (fp == NULL) {
        cout << "Arquivo nao localizado. Iniciando o programa\n";
        inicializa_array(&estoque, MAXARRAY, 0);
    } else {
        cout << "Arquivo localizado. Carregando os dados...\n";
        fscanf(fp, "%d %d", &capac, &tam);
        printf("%d registros encontrados. Capacidade atual: %d\n", tam, capac);
        inicializa_array(&estoque, capac, tam);
        
        
         if (estoque.valores == NULL) {
            cout << "Erro ao alocar a memoria para o estoque\n";
            return -2;
        }
        for (i = 0; i < estoque.tamanho; i++) {
            le_jogo_arquivo(fp, &(estoque.valores[i]));
        }
  }

    do {
        menu();
        scanf(" %c", &op);
        switch(op) {
            case 'a':
                if (adiciona_jogos(&estoque)) {
                    cout << "Item foi adicionado com sucesso!\n";
                } else {
                    cout << "Erro: Nao foi possivel adicionar.\n";
                }
                break;
            case 'b':
                // busca jogos
                 cout << "Informe o codigo: ";
                cin >> chave;
                jogos_res = busca_por_codigo(&estoque, chave);
                if(jogos_res) {
                    mostra_info(jogos_res);
                } else {
                    cout << "Item nao encontrado.\n";
                }
                break;
            case 'l':
                // lista jogos
                lista_estoque(&estoque);
                break;
            case 'r':
                // remove jogos
                cout << "Informe o codigo: ";
                cin >> chave;
                if (remove_jogos_por_codigo(&estoque, chave))
                    cout << "Item removido.\n";
                else
                    cout << "Nao foi possivel remover item.\n";
                break;
            case 'f':
                // encerra programa
                break;
            default:
                cout << "Opcao invalida!\n";
        }
    } while(op != 'f');

    //salvar as informacoes dos arquivos
    fp=fopen("ArquivoDoEstoque.txt", "w");
    if(fp== NULL) {
     	cout<<"Erro ao salvar dados\n";
     	return -1;
	 }
	fprintf(fp,"%d %d\n", estoque.capacidade, estoque.tamanho);
	
	sort(estoque.valores, estoque.valores + estoque.tamanho, ordernarPorCodigo);

	for (i = 0; i < estoque.tamanho; i++){
	    escreve_jogo_arquivo(fp, &(estoque.valores[i]));
    }
	    
    fclose(fp);
	
    return 0;
}
