https://github.com/debezium/container-images/tree/main/server/3.2

mvn clean package -DskipITs -DskipTests -Passembly

docker build --no-cache --progress=plain -t debserv-test:1.0 .