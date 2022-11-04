# LP25 - C TP n°1

Dans ce TP, vous allez utiliser `getopt` et `getopt_long` pour paramétrer le lancement d'un programme avec des options passées au programme.

## Exercice 1 - `getopt`

Dans cette partie, nous allons voir comment utiliser `getopt`, fonction qui n'admet que des options à un caractère (en plus du `-` qui les précède). La fonction getopt s'utilise en spécifiant les caractères d'options possibles, ainsi que la présence ou non d'un argument accompagnant l'option. La syntaxe est similaire au shell avec une chaîne de caractères contenant toutes les options éventuellement suivies de `:` en cas d'argument attendu.

Par exemple, le code suivant permet d'attendre l'option `-o` sans paramètre et l'option `-a` avec un paramètre. On y passe à `getopt` le nombre d'arguments à analyser, le tableau des arguments (souvent, ces deux paramètres sont ceux transmis au `main`), puis la chaîne de caractère qui définit les options attendues.

```c
int result = getopt(argc, argv, "oa:");
```

La fonction `getopt` retourne la valeur du caractère de l'option quand celle ci est rencontrée, et -1 si aucune option n'est rencontrée. Pour exploiter les options ci dessus, il est nécessaire de boucler sur l'appel à `getopt` tant que sa valeur de retour n'est pas nulle.

```c
#include <unistd.h>

int main(int argc, char *argv[]) {
	int opt = 0;
	while((opt = getopt(argc, argv, "oa:")) != -1) {
		switch (opt) {
			case 'o':
				/* Do something with 'o' flag */
				break;
				
			case 'a':
				/* Do something with 'a' and the argument pointed by optarg */
				break;
		}
	}
	return 0;
}
```

**Tâche 1**

Écrire un programme `simple-getopt.c` qui reprend le main ci-dessus et affiche les options qui ont été passées au programme (et leurs arguments éventuels le cas échéant). Jouer un peu avec les options (entrer des options non prévues, ne pas mettre d'argument là où il est requis, etc.) et observer le comportement du programme.

Il est également possible de spécifier un argument facultatif avec le caractère de l'option suivi de `::`. Dans ce cas, soit l'argument n'existe pas, soit il est dans le même argument (exemple : `-barg` pour l'option `b` et la valeur d'argument `arg`), soit il suit l'option (élément suivant de `argv`) mais il faut alors tester soi même la valeur existante dans argv (grâce à l'indice interne `optind`, en s'assurant auparavant qu'il ne dépassera pas les indices autorisés pour `argv`, i.e. `argc-1`)

**Tâche 2**

Écrire le programme `getopt-optionalarg.c` qui reprend les options de `simple-getopt.c` et y ajoute l'option `b` avec un argument facultatif. Tester les cas suivants pour vous assurer que l'implémentation est correcte :

```bash
 ./getopt-optionalarg -o -a azer -b 5214
 ./getopt-optionalarg -o -a azer -b
 ./getopt-optionalarg -breza -a 123
 ./getopt-optionalarg -b qsdf -o
```

Et assurez vous que les valeurs pour `b` sont :

 - "_5214_"
 - rien
 - "_reza_"
 - "_qsdf_"

## Exercice 2 - `getopt_long`

Il est également possible de fournir au programme des options longues préfixées par un double tiret `--`. Dans ce cas, la configuration passe par un tableau de structures de type `struct option`. Pour bien comprendre le fonctionnement, lire attentivement le manuel de la fonction `getopt_long`.

Pour expliciter son fonctionnement, nous allons remplacer les options courtes des exercices précédents par des options longues que nous nommerons `org` pour le `-o`, `app` pour le `-a` et `binary` pour le `-b` (ne cherchez pas un sens aux options, il n'y en a en l'occurrence absolument aucun). Le programme deviendrait ceci :

```c
#include <unistd.h>

int main(int argc, char *argv[]) {
	int opt = 0;
	struct option my_opts[] = {
		{.name="org",.has_arg=0,.flag=0,.val='o'},
		{.name="app",.has_arg=1,.flag=0,.val='a'},
		{.name="binary",.has_arg=2,.flag=0,.val='b'},
		{.name=0,.has_arg=0,.flag=0,.val=0}, // last element must be zero
	};
	while((opt = getopt_long(argc, argv, "", my_opts, NULL)) != -1) {
		switch (opt) {
			case 'o':
				/* Do something with 'o' flag */
				break;
				
			case 'a':
				/* Do something with 'a' and the argument pointed by optarg */
				break;
				
			case 'b':
				/* Do something with 'a' and the optional argument pointed by optarg */
				break;
		}
	}
	return 0;
}
```

Pour cet exercice, vous devez écrire un programme qui analyse les options longues suivantes :

- `--verbose` de valeur `'v'` sans argument, qui active le mode verbeux du programme.
- `--data-source` de valeur `'d'` avec argument, qui affectera la valeur en argument à une variable qui définit une source de données fictive.
- `--temporary-directory` de valeur `'t'` avec argument, qui affectera la valeur en argument à une variable qui définit le répertoire temporaire de traitement des données.
- `--output-file` de valeur `'o'` avec argument, qui affectera la valeur en argument à une variable qui définit le nom du fichier de sortie de l'application.
- `--cpu-multiplier` de valeur `'c'` avec argument, qui affectera la valeur en argument à une variable qui définit un nombre de tâches par thread du processeur de la machine.

Pour tester le programme, vous analyserez les arguments du programme à l'aide de `getopt_long`, puis, une fois cette analyse terminée, vous afficherez les valeurs des paramètres de la manière suivante :

```text
Verbose mode is [on/off] #on ou off selon la variable
Data source is located on ... #... remplacés par le chemin de data_source
Temporary files will be located on ... #... remplacés par le chemin de temporary_directory
Output file will be produces on ... #... remplacés par le chemin de output_file
N tasks will run in parallel # où N est égal à cpu_multiplier * nb_cpu
```

Le nombre de threads de l'ordinateur peut être obtenu avec la fonction `get_nprocs()` du fichier d'entête `sys/sysinfo.h`

## Exercice 3 - struct, union et mémoire

Dans ce programme, nous allons reprendre l'exercice 2 mais cette fois, la configuration va être stockée dans une structure.

Pour cela, utilisez le modèle suivant :

```c
#define STR_MAX 1024
#define PARAMETER_BUFFER_SIZE 16

typedef enum {
	PARAM_VERBOSE, /* Verbose parameter */
	PARAM_DATA_SOURCE, /* Data source parameter */
	PARAM_TEMP_DIR, /* Temp directory parameter */
	PARAM_OUTPUT_FILE, /* Output file parameter */
	PARAM_CPU_MULT, /* CPU multiplier parameter */
} parameter_id_t;

typedef union {
	long long int_param; // For CPU mult
	bool flag_param; // For verbose
	char str_param[STR_MAX]; // For path: data, temp dir & output
} data_wrapper_t;

typedef struct {
	parameter_id_t parameter_type;
	data_wrapper_t parameter_value;
} parameter_t;

int main(int argc, char *argv[]) {
	parameter_t my_parameters[PARAMETER_BUFFER_SIZE];
	int parameters_count = 0; // Increment when receiving a param
	parameter_t default_parameters[] = {
		{.parameter_type=PARAM_VERBOSE, .parameter_value.flag_param=false},
		{.parameter_type=PARAM_DATA_SOURCE, .parameter_value.str_param=""},
		{.parameter_type=PARAM_TEMP_DIR, .parameter_value.str_param="./temp"},
		{.parameter_type=PARAM_OUTPUT_FILE, .parameter_value.str_param=""},
		{.parameter_type=PARAM_CPU_MULT, .parameter_value.int_param=2},
	};
	int default_parameters_count = sizeof(default_parameters) / sizeof(parameter_t);
	/* Do stuff here */
	return 0;
}
```

Le tableau des paramètres par défaut permet de donner des valeurs par défaut aux paramètres du programme. Vous remplirez ensuite le tableau des paramètres `my_parameters` en tenant un compte des paramètres renseignés dans la variable `parameters_count` (qui définit également l'élément courant du tableau).

Une fois la vérification des paramètres effectuée, vous pourrez fusionner les deux tableaux de paramètres en écrasant les valeurs par défaut par les nouvelles valeurs issues de l'analyse des arguments du programme.

Vous écrirez ensuite la fonction suivante :
```c
bool are_parameters_valid(parameter_t *params, int params_count);
```

Cette fonction vérifie que les paramètres sont acceptables, c'est à dire :

- aucun des chemins (data source, temp dir, output file) n'est une chaîne vide;
- la valeur du multiplicateur de CPU est comprise entre 1 et 4 (bornes incluses).

La fonction renvoie `true` si les paramètres sont acceptables, `false` sinon.

## Exercice 4 - buffers circulaires

Dans cet exercice nommé `circular.c`, nous allons écrire le code d'un buffer circulaire avec un masque d'accès. L'objectif est de lire au clavier des phrases écrites par l'utilisateur et de les traiter pour mettre les caractères en majuscules. Pour cela, le programme entre dans une boucle infinie, dans laquelle il demande à l'utilisateur de saisir du texte. Puis il copie ce texte caractère par caractère dans le buffer circulaire. Quand la saisie est terminée, le programme lit tout ce qui peut être lu dans le buffer et l'affiche en mettant si possible (i.e. quand les caractères sont des lettres) le caractère en majuscule.

Un buffer circulaire est un buffer de taille fixe qui se remplit en avançant un index d'écriture et se vide dans la même direction (index croissant) en avançant un index de lecture. La lecture est impossible quand le buffer est vide (`index lecture == index écriture`). L'écriture est impossible quand le buffer est plein (`index écriture + 1 == index lecture`). Les index "bouclent", c'est-à-dire qu'arrivé au bout de la mémoire du buffer, un index repart à zéro.

Le bouclage peut être fait de deux manières : soit avec un calcul de modulo, soit avec un masque. Le calcul de modulo est un peu plus complexe (il faut tester la valeur du modulo du nouvel index) mais permet de traiter n'importe quelle taille de buffer. Le bouclage avec masque est plus simple mais il ne fonctionne que sur des tailles de buffers en puissances de 2 (i.e. 2, 4, 8, 16, 32, 64, etc.). C'est ce deuxième choix que nous allons utiliser.

Le principe des indices avec masques est d'utiliser le dépassement de taille des variables à notre avantage. En prenant par exemple une taille de buffer de 16, représentable par 4 bits et des indices de 0 (soit 0x0 ou 0000b) à 15 (soit 0xf ou 1111b), il est facile de revenir de 15 à zéro par une incrémentation assortie d'un masque. En effet, `15 + 1` étant égale à 16, soit 10000b, appliquer un masque de 15, soit 1111b à cette valeur (avec l'opération `index & 0x0f`) retournera zéro. Ceci fonctionne même en atteignant les limites en mémoire de la variable qui contient l'index. Par exemple, avec un `unsigned short`, et toujours un buffer de taille 16, lorsque la valeur atteint 65535, soit 11111111 11111111b, l'incrémentation de la valeur va créer un dépassement de valeur et réinitialiser la valeur de l'indice à 0 (car `(unsigned short) (65535 + 1)` est égal à zéro).

Vous vous appuierez sur le code suivant pour réaliser cet exercice :

```c
#define MASK 0x7f // Index mask from 0 to 127, i.e. 128 elements
#define BUFFER_SIZE 128
unsigned char read_index = 0;
unsigned char write_index = 0;
char circular_buffer[BUFFER_SIZE]

int write_char(char c);
int read_char(char *c);

int main(int argc, char *argv[]) {
	/* Do stuff here */
	return 0;
}

/*!
 * \brief function write_char writes c to circular buffer unless it is full
 * \param c the character to be appended to circular buffer
 * \return 1 if insertion is successful, 0 if buffer is full
 */
int write_char(char c) {
	/* Implement function here */
	return 1;
}

/*!
 * \brief function read_char reads a character from circular
 * \param c a pointer to the character to be read
 * \return 1 if successful, 0 if buffer is empty
 */
int read_char(char *c) {
	/* Implement function here */
	return 1;
}
```

## Bilan

Dans ce TP, vous avez eu l'occasion de vous familiariser avec les notions et fonctions suivantes :

- `getopt` pour l'analyse des options passées à l'appel de votre programme
- `getopt_long` pour l'analyse des options, avec des options longues (options de plus d'un caractère précédés par un double tiret `--`)
- les structures `struct` pour construire des données structurées
- les unions `union` pour construire des types variables (i.e. pouvant contenir alternativement n'importe quel membre mais un seul à la fois)
- les énumérations `enum` pour définir des plages de valeurs limitées
- les limites des tailles de données avec les valeurs de dépassement des types numériques
- les masques binaires pour limiter la lecture à une sous-partie des bits d'une variable.
