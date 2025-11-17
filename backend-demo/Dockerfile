# ----------------------
# 1) BUILD STAGE MAVEN
# ----------------------
FROM maven:3.9.6-eclipse-temurin-17 AS builder

WORKDIR /app

# Copier le fichier pom.xml puis télécharger les dépendances
COPY pom.xml .
RUN mvn -B dependency:go-offline

# Copier le code source
COPY src ./src

# Construire l’application
RUN mvn -B clean package -DskipTests


# ----------------------
# 2) RUNTIME STAGE
# ----------------------
FROM eclipse-temurin:17-jdk

WORKDIR /app

# Copier le jar depuis le build Maven
COPY --from=builder /app/target/*.jar app.jar

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "app.jar"]
