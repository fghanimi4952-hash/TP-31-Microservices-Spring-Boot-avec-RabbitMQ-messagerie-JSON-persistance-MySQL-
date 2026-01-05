# Architecture Microservices - Messaging avec RabbitMQ

## 📋 Description

Ce projet démontre l'implémentation d'une architecture microservices utilisant RabbitMQ pour la messagerie asynchrone. Il comprend deux microservices principaux qui communiquent via un message broker :

- **Producer Service** : Service qui produit et envoie des messages (utilisateurs) via RabbitMQ
- **Consumer Service** : Service qui consomme les messages et les persiste dans une base de données MySQL

## 🏗️ Architecture

```
┌─────────────────┐         ┌──────────────┐         ┌─────────────────┐
│  Producer       │────────▶│  RabbitMQ    │────────▶│  Consumer       │
│  (Port 8081)    │  Send   │  (Broker)    │  Queue  │  (Port 8080)    │
│                 │         │              │         │                 │
└─────────────────┘         └──────────────┘         └─────────┬───────┘
                                                               │
                                                               ▼
                                                       ┌──────────────┐
                                                       │   MySQL      │
                                                       │  Database    │
                                                       └──────────────┘
```

### Flux de données

1. Le **Producer Service** reçoit une requête HTTP POST avec un objet `User`
2. Le message est envoyé à RabbitMQ via un exchange et une routing key
3. Le **Consumer Service** écoute la queue et reçoit le message
4. Le message est automatiquement persisté dans la base de données MySQL

## 🛠️ Technologies utilisées

- **Java 21**
- **Spring Boot 4.0.1**
- **Spring AMQP** (pour l'intégration RabbitMQ)
- **RabbitMQ** (message broker)
- **MySQL** (base de données)
- **Maven** (gestion des dépendances)
- **Lombok** (réduction du code boilerplate)

## 📦 Prérequis

Avant de démarrer le projet, assurez-vous d'avoir installé :

- **Java 21** ou supérieur
- **Maven 3.6+**
- **RabbitMQ Server** (en cours d'exécution)
- **MySQL Server** (en cours d'exécution)

### Installation de RabbitMQ

#### macOS (avec Homebrew)
```bash
brew install rabbitmq
brew services start rabbitmq
```

#### Docker
```bash
docker run -d --name rabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:3-management
```

L'interface de gestion est accessible sur : http://localhost:15672 (guest/guest)

### Installation de MySQL

#### macOS (avec Homebrew)
```bash
brew install mysql
brew services start mysql
```

#### Docker
```bash
docker run -d --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=root -e MYSQL_DATABASE=testdb mysql:8.0
```

## 🚀 Installation et Configuration

### 1. Cloner le projet
```bash
git clone <repository-url>
cd micr-TP31
```

### 2. Configuration de la base de données

Créez la base de données MySQL :
```sql
CREATE DATABASE testdb;
```

### 3. Configuration RabbitMQ

Par défaut, les services utilisent :
- **Host** : localhost
- **Port** : 5672
- **Username** : guest
- **Password** : guest
- **Exchange** : user.exchange
- **Queue** : user.queue
- **Routing Key** : user.routingkey

Ces paramètres peuvent être modifiés dans les fichiers `application.yaml` de chaque service.

### 4. Compilation du projet

```bash
# Compiler le Producer Service
cd microservices-messaging-producer
mvn clean install

# Compiler le Consumer Service
cd ../microservices-messaging-consumer
mvn clean install
```

## ▶️ Démarrage

### Démarrer les services

**Terminal 1 - Consumer Service** (doit être démarré en premier)
```bash
cd microservices-messaging-consumer
mvn spring-boot:run
```
Le service démarre sur le port **8080**

**Terminal 2 - Producer Service**
```bash
cd microservices-messaging-producer
mvn spring-boot:run
```
Le service démarre sur le port **8081**

## 📡 Utilisation

### Envoyer un message (User)

Utilisez l'API REST du Producer Service pour envoyer un message :

```bash
curl -X POST http://localhost:8081/api/produce \
  -H "Content-Type: application/json" \
  -d '{
    "userId": "123",
    "userName": "John Doe"
  }'
```

### Réponse attendue

```json
"user sent: User(userId=123, userName=John Doe)"
```

### Vérification

1. **Logs du Consumer** : Vous devriez voir dans les logs du Consumer Service :
   ```
   persisted User(userId=123, userName=John Doe)
   User recieved: User(userId=123, userName=John Doe)
   ```

2. **Base de données MySQL** : Vérifiez que l'utilisateur a été sauvegardé :
   ```sql
   SELECT * FROM user;
   ```

3. **Interface RabbitMQ** : Accédez à http://localhost:15672 pour visualiser les queues et les messages

## 📁 Structure du projet

```
micr-TP31/
├── microservices-messaging-producer/     # Service Producteur
│   ├── src/main/java/
│   │   └── microservices_messaging_producer/
│   │       ├── config/
│   │       │   └── RabbitMQConfig.java   # Configuration RabbitMQ
│   │       ├── controller/
│   │       │   └── ProducerController.java
│   │       ├── domain/
│   │       │   └── User.java            # Modèle de données
│   │       ├── service/
│   │       │   └── ProducerService.java # Service d'envoi de messages
│   │       └── MicroservicesMessagingProducerApplication.java
│   └── src/main/resources/
│       └── application.yaml             # Configuration (port 8081)
│
├── microservices-messaging-consumer/      # Service Consommateur
│   ├── src/main/java/
│   │   └── microservices_messaging_consumer/
│   │       ├── config/
│   │       │   └── RabbitMQConfig.java   # Configuration RabbitMQ
│   │       ├── domain/
│   │       │   └── User.java
│   │       ├── repository/
│   │       │   └── UserRepository.java   # Repository JPA
│   │       ├── service/
│   │       │   └── ConsumerService.java  # Service de réception de messages
│   │       └── MicroservicesMessagingConsumerApplication.java
│   └── src/main/resources/
│       └── application.yaml             # Configuration (port 8080, MySQL)
│
├── spring-rabbitmq-producer/             # Exemple alternatif
├── spring-rabbitmq-consumer/             # Exemple alternatif
└── README.md
```

## 🔧 Configuration détaillée

### Producer Service (`application.yaml`)
```yaml
spring:
  rabbitmq:
    host: localhost
    username: guest
    password: guest
    port: 5672
    exchange: user.exchange
    routingkey: user.routingkey

server:
  port: 8081
```

### Consumer Service (`application.yaml`)
```yaml
spring:
  rabbitmq:
    host: localhost
    username: guest
    password: guest
    port: 5672
    exchange: user.exchange
    queue: user.queue
    routingkey: user.routingkey
  
  datasource:
    url: jdbc:mysql://localhost:3306/testdb?useSSL=false&serverTimezone=UTC
    username: root
    password: root
    driver-class-name: com.mysql.cj.jdbc.Driver
  
  jpa:
    hibernate:
      ddl-auto: update

server:
  port: 8080
```

## 🧪 Tests

Pour exécuter les tests unitaires :

```bash
# Producer Service
cd microservices-messaging-producer
mvn test

# Consumer Service
cd microservices-messaging-consumer
mvn test
```

## 📝 Modèle de données

### User
```java
public class User implements Serializable {
    private String userId;
    private String userName;
}
```

## 🔍 Dépannage

### Problème : Le Consumer ne reçoit pas les messages
- Vérifiez que RabbitMQ est en cours d'exécution
- Vérifiez que les configurations (exchange, queue, routing key) correspondent
- Vérifiez les logs des deux services

### Problème : Erreur de connexion à MySQL
- Vérifiez que MySQL est en cours d'exécution
- Vérifiez les credentials dans `application.yaml`
- Vérifiez que la base de données `testdb` existe

### Problème : Erreur de connexion à RabbitMQ
- Vérifiez que RabbitMQ est en cours d'exécution : `rabbitmqctl status`
- Vérifiez les credentials (par défaut : guest/guest)
- Vérifiez que le port 5672 est accessible

## 📚 Ressources

- [Documentation Spring AMQP](https://spring.io/projects/spring-amqp)
- [Documentation RabbitMQ](https://www.rabbitmq.com/documentation.html)
- [Spring Boot Documentation](https://spring.io/projects/spring-boot)

## 👥 Auteur

Projet réalisé dans le cadre du cours d'Architecture Microservices - TP31

## 📄 Licence

Ce projet est à des fins éducatives.
