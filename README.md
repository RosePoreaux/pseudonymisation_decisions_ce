# Introduction 

## À qui s'adresse ce guide ?

Ce guide s'adresse principalement aux organismes publics, et plus particulièrement aux personnes chargées de la mise en place de la pseudonymisation des données dans ces organismes. Il pourra également intéresser d'autres acteurs faisant face à un besoin de pseudonymisation de documents textuels. 

## A quoi sert ce guide ? 

Ce guide présente les différentes étapes d'un projet de pseudonymisation de documents à l'aide de méthodes d'Intelligence Artificielle. Il vient en complément d'un brique de code Open Source, hébergée sur GitHub [ici](https://github.com/etalab-ia/pseudonymisation_decisions_ce), rassemblant des outils clés en main pour entraîner un modèle de pseudonymisation et utiliser ce modèle afin d'automatiser la pseudonymisation de documents textuels. 


## Comment contribuer ?

Ce document est un outil évolutif et ouvert. Vous pouvez contribuer à l'améliorer en proposant une modification dans la version éditable du guide ([sur Github](https://github.com/etalab-ia/pseudonymisation_decisions_ce)) ou en contactant directement l'équipe du Lab IA d'Etalab (lab-ia@data.gouv.fr). 


# Qu'est-ce que la pseudonymisation ? 

## Quelles sont les différences entre anonymisation et pseudonymisation ?

Nous reprenons ici l'explication présentée dans le [guide de la CNIL sur l'anonymisation des données](https://www.cnil.fr/fr/lanonymisation-des-donnees-un-traitement-cle-pour-lopen-data): 

***
« La pseudonymisation est un traitement de données personnelles réalisé de manière à ce qu'on ne puisse plus attribuer les données relatives à une personne physique sans avoir recours à des informations supplémentaires. En pratique la pseudonymisation consiste à remplacer les données directement identifiantes (nom, prénom, etc.) d’un jeu de données par des données indirectement identifiantes (alias, numéro dans un classement, etc.).

La pseudonymisation permet ainsi de traiter les données d’individus sans pouvoir identifier ceux-ci de façon directe. En pratique, il est toutefois bien souvent possible de retrouver l’identité de ceux-ci grâce à des données tierces. C’est pourquoi des données pseudonymisées demeurent des données personnelles. L’opération de pseudonymisation est réversible, contrairement à l’anonymisation. » 
*** 

Pour résumer, des données pseudonymisées ne sont pas tout à fait anonymes, mais ne permettent pas non plus de réidentifier directement les personnes. La pseudonymisation a pour effet de réduire la corrélation entre les données directement identifiantes et les autres données d'une personne. 


## Pourquoi pseudonymiser ?


La [loi n°2016-1321 du 7 octobre 2016 pour une République numérique](https://www.legifrance.gouv.fr/affichLoiPubliee.do?idDocument=JORFDOLE000031589829&type=general&legislature=14)  fait de l’ouverture des données publiques la règle par défaut. Pour plus d'informations à ce sujet, vous pouvez consulter [le guide Etalab sur l'ouverture des données publiques](https://guides.etalab.gouv.fr/juridique/ouverture/#la-communication-de-vos-documents-administratifs). 

Lorsque les administrations souhaitent diffuser des documents administratifs contenant des données personnelles, l'occultation préalable des éléments à caractère personnel est une obligation légale qui s’impose à elles par principe en application du Code des relations entre le public et l’administration, [CRPA article L. 312-1-2](https://www.legifrance.gouv.fr/affichCodeArticle.do?idArticle=LEGIARTI000033205514&cidTexte=LEGITEXT000031366350&dateTexte=20161009). 

Pour satisfaire à cette obligation légale, la CNIL préconise d'anonymiser les documents administratifs avant de les diffuser. Néanmoins, pour les documents administratifs qui contiennent des données non structurées -- c'est-à-dire du texte libre -- une complète anonymisation, qui garantirait une parfaite impossibilité de réidentification, est difficile à atteindre et aboutirait à une perte trop grande d'informations. 

La diffusion des décisions de justice, sur le site Légifrance notamment, s'opère ainsi une fois leur pseudonymisation réalisée. Voici [un exemple de décision pseudonymisée](https://www.legifrance.gouv.fr/affichJuriJudi.do?oldAction=rechJuriJudi&idTexte=JURITEXT000041701871&fastReqId=757329309&fastPos=1) sur Légifrance. 



## Quelles données personnelles dois-je retirer de mon jeu de données ?

Cela dépend du contexte réglementaire, le même cadre ne s'applique pas à tous les documents.  
Néanmoins, il conviendra la plupart du temps de pseudonymiser toute information se rapportant à une personne physique identifiée ou identifiable. Une «personne physique identifiable» est une personne physique qui peut être identifiée, directement ou indirectement, notamment par référence à un identifiant, tel qu'un nom, un numéro d'identification, des données de localisation, un identifiant en ligne, ou à un ou plusieurs éléments spécifiques propres à son identité physique, physiologique, génétique, psychique, économique, culturelle ou sociale.




# Quelles méthodes de pseudonymisation ?

## Cas où les données à caractère personnel sont tabulaires

Lorsque les données à caractère personnel sont contenues dans un jeu de données structurées, il est aisé de procéder directement à des traitements visant à pseudonymiser ou anonymiser, en supprimant les colonnes concernées ou en cryptant leur contenu. Ce cas de figure n'est pas l'objet de ce guide. Pour plus d'informations à ce sujet, on se référera [aux ressources de la CNIL sur l'anonymisation](https://www.cnil.fr/fr/lanonymisation-des-donnees-un-traitement-cle-pour-lopen-data).


## Cas où les données à caractère personnel apparaissent dans du texte libre

Lorsque les données à caractère personnel sont contenues dans du texte libre, le ciblage précis des éléments identifiants dans le texte est une tâche souvent complexe. Lorsqu'elle est réalisée par des humains, cette tâche est coûteuse en temps et peut requérir une expertise spécifique à la matière traitée (textes juridiques par exemple). L'IA et les techniques de traitement du langage naturel peuvent permettre d'automatiser cette tâche souvent longue et fastidieuse. 


## Puis-je utiliser l'Intelligence Artificielle (IA) pour pseudonymiser ?

L'utilisation de l'IA pour automatiser la pseudonymisation de vos documents peut être plus ou moins pertinente. Les solutions d'IA pour pseudonymiser des données textuelles sont en grande majorité des modèles supervisés. Ces modèles d'IA dits d'apprentissage supervisés se sont beaucoup développés ces dernières années, en particulier dans le domaine du « deep learning », et sont en général les plus performants. Mais pour que ces modèles puissent afficher de bonnes performances, un certain nombre de prérequis sont à remplir, que nous détaillons dans les paragraphes de cette section. 

Il existe d'autres méthodes permettant d'automatiser la tâche de pseudonymisation, comme les moteurs de règles. Les moteurs de règles sont un ensemble de règles prédéfinies "à l'avance". Par exemple, une règle de pseudonymisation pourrait être "si le mot qui suit Monsieur ou Madame commence par une majuscule alors ce mot est un prénom". La complexité du langage naturel et la diversité des formulations qui se trouvent dans les documents fait que ce type de moteur de règles a de forte chance de faire beaucoup d'erreurs. 


Nous présentons ci-après quelques paramètres à prendre en compte pour juger de la pertinence de l'utilisation de l'IA pour pseudonymiser. 


### Données annotées 


Dans le champ de l'apprentissage automatique, les modèles supervisés sont des algorithmes qui prennent en entrée des données avec des "labels" afin qu'ils "apprennent", lorsqu'on leur présente une nouvelle donnée "non-labelisée", à lui attribuer le bon label. Dans le cas de la pseudonimisation, les labels sont les catégories (nom, prénom, adresse, etc.) que l'on attribue à chaque mot d'un document. Ces catégories varient selon la nature du document et le degré de pseudonymisation souhaité. En traitement du langage naturel, ce type de tâche s'appelle la reconnaissance d'entités nommées (*named entity recognition (NER)* en anglais). Lorsqu'elle est réalisée par un humain, la tâche consistant à attribuer des labels à certains mots ou groupes de mots d'un document s'appelle l'annotation. On parlera de labélisation pour l'attribution d'un label à un mot ou à un autre élément donné, et d'annotation pour l'attribution de différents labels à des mots d'un document. Afin de constituer un ensemble de documents annotés qui va servir à entraîner un algorithme d'IA à automatiser cette tâche, il est nécessaire d'utiliser un logiciel d'annotation qui permet d'enregistrer les différentes annotations réalisées. 

Etre en mesure d'entraîner un algorithme d'IA pour pseudonymiser dépend donc de la disponibilité de documents annotés ou de la possibilité d'annoter des documents. 

Nous avons développé [un outil d'annotation basé sur Doccano](http://0.0.0.0/) que vous pouvez tester.


### La qualité et le volume des données 

Le volume de documents annotés nécessaires dépendra de la complexité de la tâche de pseudonymisation, qui sera fonction, entre autres, du nombre de catégories d'entités nommées retenues et de la complexité du langage utilisé dans les documents. Il est en général nécessaire d'annoter de l'ordre d’un à plusieurs milliers de documents afin d'obtenir des résultats optimaux. 

La qualité des données est un autre critère essentiel qui sera déterminant pour la performance de l'algorithme. On distinguera la qualité des données textuelles brutes et la qualité des annotations réalisées. 

Les données textuelles peuvent se présenter sous différents formats, plus ou moins lisibles. Idéalement, les documents textuels sont stockés au format *txt* ou *json*. Des formats moins standards (*doc*, *pdf*, *png*, etc..) nécessiteront des conversions afin de pouvoir être traités. Lorsque les documents sont au format image (car résultant d'une numérisation de documents papiers), la mise en place d'une brique d'OCR sera nécessaire afin de les convertir au format texte. 

La qualité des annotations est également essentielle, et ce pour deux raisons principales : l'apprentissage de l'algorithme et l'évaluation de la performance de l'algorithme. 

Une partie des données annotées va en effet servir à apprendre à l'algorithme à réaliser la tâche. Des données mal annotées (omissions d'entités nommées, attribution de la mauvaise catégorie d'entité) va donc conduire l'algorithme à mal prédire les catégories des mots des nouveaux documents. Une autre partie des données va servir à évaluer la performance de l'algorithme, en comparant les labels prédits par l'algorithme à ceux déterminés "manuellement". Si les labels issus de l'annotation par des humains ne sont pas fiables, l'évaluation de la performance de l'algorithme ne sera pas fiable. 


### L'accès à des infrastructures de calcul adéquates (serveurs de stockage, RAM, GPU) 

L'apprentissage de modèles de NLP récents, basés sur des réseaux de neurones nécessite des ressources dédiées, notamment des ressources de type GPU afin d'accélérer considérablement le temps de calcul. Même en disposant de GPU de dernières générations, il faut compter plusieurs jours voire plusieurs semaines pour un apprentissage complet du modèle.


![alt text](Choice_vf.svg "Logo Title Text 1")


# Quelles sont les étapes d'un projet de pseudonymisation ?

## Les données

Les données sont constituées de l'ensemble des documents (texte libre) desquels il faut occulter des éléments identifiants. 

On distingue parmi les données les données d'entraînement, les données de test et les données à labéliser. 

Les jeux de données d'entraînement et les données de tests sont tous deux constitués de données annotées. Comme indiqué précédemment, avoir des données annotées est un prérequis à la mise en œuvre d'un modèle d'apprentissage supervisé de pseudonymisation. Ce sont les données d'entraînement, annotées au préalable par des humains, qui vont permettre à l'algorithme d'apprendre à reproduire cette tâche. Afin d'évaluer la précision de l'algorithme, il faut utiliser d'autres données annotées (non utilisée pour l'entraînement). C'est le rôle du jeu de données de test. 

Enfin, les données n'ayant pas été annotées vont pouvoir être pseudonymisées automatiquement par l'algorithme entraîné sur les données annotées.  



## L'annotation

Afin de pouvoir utiliser les données annotées pour l'entraînement d'un algorithme d'apprentissage, celles-ci doivent être converties dans un format spécifique. Dans l'exemple ci-dessous, un document textuel (ici "Thomas Clavier aime beaucoup Paris.") est alors structuré en un tableau, avec un mot par ligne, et deux colonnes, une pour le mot (ou *token*) et une pour l'annotation linguistique. 

| Token   | Label  | 
| ----------| ----------| 
| Thomas | B-PER | 
| CLAVIER | I-PER | 
| aime | O | 
| beaucoup | O | 
| Paris | B-LOC  | 


Nous utilisons le format BIO, très utilisé pour les tâches de reconnaissance d'entités nommées, pour labéliser nos données. Le préfixe B- avant un label indique que le label est le début d'un groupe de mots, le préfixe I- indique que le label est à l'intérieur d'un groupe de mots, et le label O indique que le *token* n'a pas de label particulier.  
Pour aider au formatage des données, notre outil comporte une section permettant de convertir des données au format *json* annotés par Doccano au format CONLL.  

***

> ### Le format CoNLL : 
> CoNNL, pour Conference on Natural Language Learning, est un format général, dont il existe de nombreuses versions, couramment employé pour les tâches de NLP, décrivant des données textuelles en colonne selon un nombre d'attribut (catégorie d'entité nommée, nature grammaticale, etc.). Le format BIO que nous utilisons fait partie des formats CoNLL. 
***

Il existe de très nombreux logiciels ou solutions d'annotation de données textuelles et les données annotées en sortie peuvent donc avoir différents formats (il existe en effet de multiples formats de données annotées). Pour transformer vos données annotées, un développement spécifique sera probablement nécessaire afin de les convertir au format BIO, le format des données d'entrée de l'algorithme de reconnaissance d'entités nommées. Plusieurs exemples de fonctions et de librairies développées pour le Conseil d'État constitueront néanmoins un point de départ dans le répertoire GitHub.

***
> ### Exemple de mise en œuvre : moteur de pseudonymisation pour le Conseil d'État
> Le conseil d'État dispose de 250 000 décisions provenant des différents échelons de la justice administrative, qui ont été annotées de façon automatique par un moteur de règles. Parmi celles-ci, 25 000 avaient reçu une vérification manuelle. Le système de pseudonymisation automatique par moteur de règles a un taux d'erreur élevé et nécessite une relecture manuelle afin de garantir une bonne précision de la reconnaissance d'éléments identifiants. 
Afin d'entraîner un algorithme supervisé, nous avons utilisé un sous-échantillon de 5000 décisions parmi les 25000 vérifiées à la main. Nous avons sélectionné un sous-échantillon car l'échantillon complet des données vérifiées manuellement est trop important au vue des ressources de calcul dont nous disposons pour l'entraînement du modèle.  
***

## Tokénisation

Si l'on considère un document, composé de blocs de caractères, la tokénisation est la tâche qui consiste à découper ce document en éléments atomiques, en gardant ou supprimant la ponctuation. Par exemple :  

| Phrase        | 
| :------------- |
| Mes amis, mes enfants, l'avènement de la pseudonymisation automatique est proche. | 

La phrase ci-dessus pourrait être tokenisée des manières suivantes :

| Token1   | Token2   | Token3   | Token4   | Token5   | Token6   | Token7   | Token8   | Token9   | Token10   | Token11   | Token12   | Token13   | Token14   | Token15   | Token16   |
| :----------| :----------| :----------| :----------| :----------| :----------| :----------| :----------| :----------| :----------| :----------|  :----------| :----------| :----------| :----------| :----------| 
| Mes  | amis  |  , | mes  | enfants  | ,   |l   | ' | avènement  | de   | la   |pseudonymisation| automatique| est  | proche  | .|
 

| Token1   | Token2   | Token3   | Token4   | Token5   | Token6   | Token7   | Token8   | Token9   | Token10   | Token11   |
| :----------| :----------| :----------| :----------| :----------| :----------| :----------| :----------| :----------| :----------| :----------| 
| Mes| amis| mes| enfants| l'avènement | de | la| pseudonymisation| automatique | est| proche|


Les tokens correspondent généralement aux mots, mais il est important de comprendre qu'une autre unité peut être choisie, par exemple les caractères. D'autres choix dans la façon de tokeniser peuvent avoir un impact sur les résultats de l'algorithme. Par exemple, le choix de conserver ou non la ponctuation a son importance.  
De manière pratique, il est important de bien comprendre la méthode de tokénisation utilisée par les algorithmes, afin de prendre en compte ces choix lors de l'étape finale d'occultation d'éléments identifiants dans le texte. 
Notre outil utilise les tokenisateurs du package **NLTK** : **WordPunctTokenizer** pour tokeniser une phrase en éléments, et **PunktSentenceTokenizer** pour découper le document en phrases (ou plus communément *sentences*, en anglais).


## Apprentissage

Dans le code que nous avons développé, nous utilisons la librairie Open Source [Flair](https://github.com/flairNLP/flair). 

Il est possible d'utiliser de nombreux modèles de langage supportés par Flair, et notamment les modèles Bert et Camembert, ou de combiner plusieurs modèles de langages.
Ce modèle de langage permet pour chaque mot d'obtenir une représentation vectorielle (ou *embedding*).  Cet embedding est ensuite passé à un classificateur BiLSTM-CRF qui attribue à chaque mot une des classes du jeu de données d'entrainement.

L'entrainement d'un tel classificateur nécessite de choisir la valeur d'un certain nombre d'hyper-paramètres (les hyper-paramètres sont les paramètres de l'algorithmes qui sont fixés avant l'apprentissage, par opposition aux paramètres de l'algorithmes qui sont fixés de manière itérative au cours de l'apprentissage). Des exemples de configuration avec des explications des différents hyper-paramètres et de leur impact sont disponibles dans la section correspondante du répertoire GitHub.  
# à rajouter ici : lien ver le fichier correspondant dans le repo 

Nous proposons un exemple de module permettant d'entrainer un algorithme de reconnaissance d'entités nommées via la librairie FLAIR à partir d'un corpus annoté.  
Enfin, pour aller plus loin, la librairie Flair propose [un module très pratique permettant de fixer les valeurs optimales des hyper-paramètres optimaux pour l'apprentissage](https://github.com/flairNLP/flair/blob/master/resources/docs/TUTORIAL_8_MODEL_OPTIMIZATION.md).


## Validation

La validation des performances du modèle et la mise en production est un double processus reposant sur l'analyse de métriques et sur l'expérience humaine.

![alt text](process.svg "Logo Title Text 1")


Notre module de génération de documents pseudonymisés permet de produire en sortie des fichiers labélisés, au format CoNLL. Ceci permet de :   

 - Utiliser des métriques permettant de comparer, pour un document ayant été annoté manuellement, la pseudonymisation par l'algorithme à celle réalisée manuellement. On utilise généralement le [F1-score](https://fr.wikipedia.org/wiki/Pr%C3%A9cision_et_rappel) pour mesurer la performance du modèle.
 - Charger dans [notre outil d'annotation basé sur Doccano](http://0.0.0.0/) un fichier mettant en avant les différences entre les annotations provenant de sources différentes, indiquant en rouge les labels en désaccord et en vert les labels en accord.


## Génération du document pseudonymisé

Le modèle entrainé permet d'attribuer une catégorie à chaque token du document à pseudonymiser. Les sorties de l'algorithme de reconnaissance d'entités nommées ne permettent donc pas d'obtenir directement le document peudonymisé (texte original dans lequel les éléments à caractère personnel ont été remplacés par des alias). Pour le bon fonctionnement de cette étape, il est très important de fournir à l'algorithme un document tokénisé selon une méthode identique à celle utilisée pour entrainer l'algorithme.  
Générer un document pseudonymiser nécessite de reconstruire le texte orginal à partir des sorties de l'algorithme : notre outil propose un module permettant de tester la performance de l'algorithme de reconnaissance d'entités nommées fourni nativement par Flair, ou un modèle entrainé sur des données spécifiques, et de générer des documents pseudonymisés. Le résultat est aussi visible [sur notre site](http://127.0.0.1:8050/).


# Quelles sont les autres ressources disponibles pour pseudonymiser 

## Les librairies

De nombreuses librairies OpenSource permettent d'entrainer et d'utiliser des algorithmes de reconnaissance d'entités nommées. Parmi celles-ci, [Flair](https://github.com/flairNLP/flair) et [Spacy](https://spacy.io/usage/spacy-101) présentent l'avantage de proposer des algorithmes à l'état de l'art tout en facilitant l'expérience utilisateur.

 - Flair: Un framework simple pour le NLP. Flair permet d'utiliser des modèles de NLP à l'état de l'art sur des textes de tout genre, en particulier des algorithmes de reconnaissance d'entité nommées et des embeddings pré-entrainés
 - SpaCy: Un framework Python à forte capacité d'industrialisation pour le NLP. Il s'agit d'une librairie pour le NLP en Python et Cython. Il implémente les toutes dernières recherches dans le domaine du traitement du langage naturel et a été conçu pour être utilisé en production. Il possède des modèles statistiques et des embeddings pré-entrainés.

Flair est la librairie que nous avons choisie pour le développement de l'outil de pseudonymisation présenté ici. 


## Annotation

[Notre outil d'annotation](http://0.0.0.0/) est basé sur Doccano.
[Doccano](https://github.com/doccano/doccano) est un outil d'annotation de texte open source. Il fournit des fonctionnalités d'annotation pour la classification de texte, la labélisation de mots et d'autres tâches classiques de NLP. Ainsi, il est possible de créer des données labélisées pour l'analyse des sentiments, **la reconnaissance d'entités nommées**, la synthèse de texte, etc.  
Il est possible de créer rapidement un jeu de données de documents labélisés en installant Doccano. Notre outil permet de transformer simplement les données annotées via Doccano en fichiers CoNLL.
De nombreux autres logiciels d'annotation sont disponibles, dont beaucoup sont Open Source. 


# Voir la pseudonymisation en action

Vous pouvez essayer notre démonstrateur de pseudonymisation sur http://127.0.0.1:8050/ ou directement voir le code sur https://github.com/etalab-ia


# Ressources
- Guide RGPD du développeur de la CNIL : https://www.cnil.fr/fr/guide-rgpd-du-developpeur
- Guide de l'anonymisation de la CNIL: https://www.cnil.fr/fr/lanonymisation-des-donnees-un-traitement-cle-pour-lopen-data


