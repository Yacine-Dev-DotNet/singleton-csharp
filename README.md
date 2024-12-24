Comprendre et implémenter le Design Pattern Singleton en C#

1. Décorticage du mot « Singleton » et origine :
   
Le terme Singleton vient de l’anglais « single », qui signifie « unique », renforcé par le suffixe « -ton » évoquant la notion de singularité (inspiré de « monoton », signifiant « un seul ton »). Dans le contexte de la programmation, le pattern Singleton a pour but de garantir qu’une classe ne puisse avoir qu’une seule instance dans une application, avec un accès global à cette instance.

Sur le plan historique, le concept du Singleton a été popularisé par le célèbre livre Design Patterns: Elements of Reusable Object-Oriented Software (1994), écrit par les « Gang of Four » (Erich Gamma, Richard Helm, Ralph Johnson et John Vlissides). Depuis, le Singleton est l’un des premiers motifs de conception (design patterns) enseignés aux développeurs pour maîtriser la gestion et le partage de ressources dans les applications.

2. Scénarios réels où utiliser le Singleton :
   
Le Singleton s’avère extrêmement pratique lorsque une seule instance d’un objet doit coordonner des actions globales ou gérer des ressources partagées à l’échelle de l’application. Parmi les exemples concrets :

2.1. Gestion de journalisation (Logging) : Un service de journalisation doit être unique pour éviter les conflits d’accès aux fichiers de logs et centraliser l’écriture des messages.

2.2. Gestion des connexions à une base de données : Un pool de connexions peut être centralisé pour optimiser les performances et éviter des ouvertures/redondances inutiles.

2.3. Configuration partagée : Avoir une instance unique pour stocker des configurations globales d’une application (paramètres d’environnement, chemins d’accès, etc.).

2.4. Gestionnaire de threads (Thread Pool) : Un Singleton peut coordonner l’exécution des threads en gérant la file d’attente et la répartition de la charge.

2.5. Service de cache global : Stocker des données fréquemment utilisées dans une instance unique pour un accès rapide (améliore les performances et la cohérence).

3. Principales étapes du pattern Singleton en C# :
   
Dans cette section, nous allons passer en revue les points clés pour créer un Singleton robuste en C#. L’idée principale est de garantir l’unicité de l’instance et d’encapsuler correctement sa création.

3.1. Rendre le constructeur privé : La première étape est de rendre le constructeur privé pour empêcher l’instanciation de la classe depuis l’extérieur. Seule la classe elle-même peut créer son instance.

        public class MyClassName
        {
            private MyClassName()
            {
                // Constructeur privé
            }
        }

Personne à l’extérieur ne peut faire

        new MyClassName().

Cela encapsule la logique de création de l’instance pour la centraliser dans la classe elle-même.

3.2 Créer la variable statique pour stocker l’instance
On déclare ensuite une variable statique (souvent _instance) qui contiendra la référence unique :

        private static MyClassName _instance;

Le risque, c’est qu’avant que _instance soit initialisé, on obtienne une NullReferenceException si on l’utilise directement. Deux stratégies s’offrent à nous :

1. Early Initialization (initialisation hâtive)
On initialise _instance dès la déclaration.

        private static MyClassName _instance = new MyClassName();

Avantage : simplicité (pas besoin de vérifier null).
Inconvénient : l’instance est créée même si l’on ne s’en sert jamais.
2. Lazy Initialization (initialisation paresseuse)
On n’instancie l’objet que lorsqu’on en a besoin, en vérifiant null dans la propriété d’accès.

        private static MyClassName _instance;
        
        public static MyClassName Instance
        {
            get
            {
                if (_instance == null)
                {
                    _instance = new MyClassName();
                }
                return _instance;
            }
        }

Avantage : création seulement lors de la première utilisation.
Inconvénient : non thread-safe dans cette version simple.

3.3. Rendre l’accès thread-safe
En environnement multi-threads, deux threads pourraient passer le if (_instance == null) en même temps et créer deux instances. Pour éviter cela, on utilise un verrou (lock) :

        private static MyClassName _instance;
        private static readonly object _lock = new object();

        public static MyClassName Instance
        {
            get
            {
                lock (_lock)
                {
                    if (_instance == null)
                    {
                        _instance = new MyClassName();
                    }
                    return _instance;
                }
            }
        }

Le premier thread qui obtient le lock crée l’instance si _instance == null.
Les autres threads attendent, puis voient _instance déjà initialisé.
Double-checked locking
Pour des raisons de performance, on peut éviter d’entrer dans le lock si _instance n’est pas null :

        public static MyClassName Instance
        {
            get
            {
                if (_instance == null)
                {
                    lock (_lock)
                    {
                        if (_instance == null)
                        {
                            _instance = new MyClassName();
                        }
                    }
                }
                return _instance;
            }
        }

Attention toutefois : la réorganisation des instructions en C# peut introduire des subtilités ; la solution la plus moderne consiste souvent à utiliser Lazy<T>.

3.4 Utiliser Lazy<T> (Option recommandée)
Depuis C# 4.0, le framework propose la classe Lazy<T> qui gère la thread-safety et l’initialisation paresseuse :

        public sealed class MyClassName
        {
            private static readonly Lazy<MyClassName> _instance =
                new Lazy<MyClassName>(() => new MyClassName());
        
            private MyClassName() { }
        
            public static MyClassName Instance
            {
                get { return _instance.Value; }
            }
        }
        
Lazy<T> est thread-safe par défaut.
L’instance est créée uniquement au premier accès (.Value).

3.5 Propriété publique pour accéder à l’instance

Pour accéder à notre Singleton, on expose souvent une propriété Instance (ou GetInstance()) :

        public static MyClassName Instance => _instance;

ou

        public static MyClassName Instance
        {
            get { return _instance; }
        }

C’est le point d’entrée officiel pour obtenir la référence unique.

3.6 Différence entre Singleton et classe statique :

On pourrait se demander : « Pourquoi ne pas faire directement une classe statique ? »
Certes, une classe statique et un Singleton ont en commun d’être « uniques » dans l’application, mais :

3.6.1. Instanciable vs. non-instanciable :

Une classe statique ne peut pas être instanciée, tout est statique.
Un Singleton reste un objet, ce qui permet de le passer en paramètre, de l’implémenter avec une interface, d’hériter, etc.

3.6.2. Interfaces et polymorphisme :

Une classe statique ne peut pas implémenter d’interface.
Un Singleton peut implémenter des interfaces et être injecté via un conteneur d’injection de dépendances, facilitant les tests unitaires.

3.6.3. Cycle de vie géré ou non :

Une classe statique « vit » tant que l’application tourne.
Un Singleton peut être géré par un conteneur DI, être recyclé ou reconfiguré.

3.6.4. Contrôle plus fin :

Le Singleton peut décider comment et quand il se crée.
La classe statique n’offre pas cette flexibilité.

3.6.5. Testabilité (mock, dépendances) :

- Les membres statiques sont difficiles à moquer.
- Un Singleton peut être mocké (fake) en test unitaire si on passe par une interface.

En résumé, la classe statique est adaptée aux utilitaires « purs » sans état ; le Singleton est préférable pour gérer un état interne, injecter des dépendances ou faciliter les tests.

4. Exemple simple de Singleton en C# :
   
Voici un exemple minimaliste montrant le principe du Singleton en C#. Dans cet exemple, on utilise la forme « lazy initialization » non thread-safe (simple) :

        public class Singleton
        {
            private static Singleton _instance;
        
            private Singleton() { }
        
            public static Singleton Instance
            {
                get
                {
                    if (_instance == null)
                    {
                        _instance = new Singleton();
                    }
                    return _instance;
                }
            }
        
            public void DoSomething()
            {
                Console.WriteLine("Singleton instance is working!");
            }
        }

Si tu souhaites le rendre thread-safe, ajoute simplement le mécanisme de verrou (lock) ou utilise Lazy<Singleton>.

Pour l’utiliser, on peut écrire :

        class Program
        {
            static void Main(string[] args)
            {
                // Récupération de l'unique instance
                Singleton single = Singleton.Instance;
                single.DoSomething();
            }
        }

5. Récapitulatif et conclusion :

1. Constructeur privé : empêche toute instanciation externe.
2. Variable statique _instance : stocke l’instance unique, accessible dans toute l’application.
3. Propriété publique : Instance ou GetInstance(), qui fournit le point d’accès officiel.
4. Gestion de la simultanéité : lock, double-checked locking ou Lazy<T> pour éviter plusieurs créations en multi-threads.
5. Différence avec une classe statique : un Singleton est un objet (implémentation d’interfaces, injection, tests unitaires possibles).

En conclusion, le Singleton est un pattern très utile pour partager et centraliser des ressources dans une application. Il faut toutefois rester vigilant quant à son utilisation : un usage excessif peut créer des dépendances globales difficiles à maintenir ou à tester. Lorsque tu souhaites un cycle de vie contrôlé, un état interne et la capacité à injecter des dépendances, le Singleton, combiné avec un conteneur d’injection (comme dans ASP.NET Core), est un excellent choix. Sinon, une classe statique (pour de simples méthodes utilitaires) peut largement suffire.

Félicitations, tu as désormais toutes les clés en main pour implémenter et comprendre en profondeur le pattern Singleton en C# ! N’hésite pas à combiner ce savoir avec d’autres patterns et à t’appuyer sur un conteneur d’injection de dépendances pour rendre ton code encore plus modulaire et maintenable.



Ressources additionnelles :
- Design Patterns: Elements of Reusable Object-Oriented Software par Erich Gamma, Richard Helm, Ralph Johnson, John Vlissides (Gang of Four).
- Documentation Microsoft sur Lazy<T>.
- Tout conteneur d’injection de dépendances (Unity, Autofac, etc.) pour gérer le cycle de vie de tes Singletons.

Article rédigé par Yacine Amirech.
