
----------DEBEZIUM----------


- Clone git
git clone --branch v2.4.0.Final --depth 1 \
  https://github.com/debezium/debezium.git \
  debezium-2.4.0.Final

cd debezium-2.4.0.Final
git describe --tags --exact-match


- Cài java 11
apt update
apt install -y openjdk-11-jdk
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
export PATH="$JAVA_HOME/bin:$PATH"
java -version
javac -version
echo "$JAVA_HOME"

- Cài mnv
apt install -y maven

- build riêng
./mvnw clean install   -pl debezium-connector-mysql   -am   -Dquick

./mvnw package \
  -pl debezium-connector-mysql \
  -am \
  -Passembly \
  -Dquick

tìm file 
 find debezium-connector-mysql/target -type f   -name '*plugin.tar.gz'


---------TRINO----------


- Clone source
git clone --branch 455 --depth 1 \
  https://github.com/trinodb/trino.git trino-455

- Cài jdk-22
sudo apt install jdk-22
- Build
./mvnw -DskipTests -Dair.check.skip-all=true clean install

- Giải nén thư mục run
mkdir -p ~/start/trino/run

tar -xzf \
  ~/start/trino/trino-455/core/trino-server/target/trino-server-455.tar.gz \
  -C ~/start/trino/run

cd ~/start/trino/run/trino-server-455

- Tạo cấu hình
mkdir -p etc/catalog data

cat > etc/node.properties <<EOF
node.environment=local
node.id=trino-local-1
node.data-dir=$HOME/start/trino/run/trino-server-455/data
EOF

cat > etc/jvm.config <<'EOF'
-server
-Xmx2G
-XX:+UseG1GC
-XX:G1HeapRegionSize=32M
-XX:+ExplicitGCInvokesConcurrent
-XX:+HeapDumpOnOutOfMemoryError
-XX:+ExitOnOutOfMemoryError
EOF

cat > etc/config.properties <<'EOF'
coordinator=true
node-scheduler.include-coordinator=true
http-server.http.port=18081
discovery.uri=http://127.0.0.1:18081
EOF

cat > etc/catalog/memory.properties <<'EOF'
connector.name=memory
EOF

- RUN
./bin/launcher run




----------OPENSEARCH----------

- Clone source
git clone --branch 2.13.0 --depth 1 https://github.com/opensearch-project/OpenSearch.git opensearch/OpenSearch-2.13.0

- Cài java 11
sudo apt update
sudo apt install -y openjdk-11-jdk
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
export PATH="$JAVA_HOME/bin:$PATH"
java -version

- build
cd Opensearch-2.13.0
./gradlew localDistro
 
- Tạo user ở ngoài vì openseach không dùng root để chạy được 
useradd --system --create-home --shell /bin/bash opensearch 2>/dev/null || true

mkdir -p /opt/opensearch

cp -a \
/root/start/opensearch/OpenSearch-2.13.0/distribution/archives/linux-tar/build/install/opensearch-2.13.0-SNAPSHOT/. \
/opt/opensearch/

chown -R opensearch:opensearch /opt/opensearch

- Run
sudo -u opensearch -H bash -c '
cd /opt/opensearch
OPENSEARCH_JAVA_OPTS="-Xms1g -Xmx1g" ./bin/opensearch
'

----------Airflow----------
Airflow

- Clone source
git clone --branch 2.7.3 --depth 1 https://github.com/apache/airflow.git airflow/airflow-2.7.3

- Cài python 3.11
sudo apt install -y python3 python3-venv python3-dev build-essential libffi-dev libssl-dev
apt install -y python3.11 python3.11-venv python3.11-dev

- Tạo visual riêng
cd /root/start/airflow

python3.11 -m venv .venv
source .venv/bin/activate

python --version
pip install --upgrade pip setuptools wheel

AIRFLOW_VERSION=2.7.3
PYTHON_VERSION=3.11
CONSTRAINT_URL="https://raw.githubusercontent.com/apache/airflow/constraints-${AIRFLOW_VERSION}/constraints-${PYTHON_VERSION}.txt"

python -m pip install "apache-airflow==${AIRFLOW_VERSION}" \
  --constraint "${CONSTRAINT_URL}"

- Đặt thư mục đặt dữ liệu 
export AIRFLOW_HOME=/root/start/airflow/airflow-home
mkdir -p "$AIRFLOW_HOME"

- Khởi tạo database
airflow db migrate

- Tạo tài khoản quản trị
airflow users create \
  --username admin \
  --password admin \
  --firstname Hai \
  --lastname Nguyen \
  --role Admin \
  --email admin@example.com

- Chạy webserrver 
airflow webserver --port 8080

- Chạy shecdukle
cd /root/start/airflow
source .venv/bin/activate
export AIRFLOW_HOME=/root/start/airflow/airflow-home

airflow scheduler



----------OPENMETADTA----------
OPENMETADATA

- clone source

OPENMETADATA_VERSION=1.4.8
git clone --branch "$OPENMETADATA_VERSION" --depth 1 https://github.com/open-metadata/OpenMetadata.git openmetadata

- Cài jdk-17
sudo apt update
sudo apt install -y openjdk-17-jdk maven nodejs npm
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
export PATH="$JAVA_HOME/bin:$PATH"
java -version
mvn -version
node -v
npm -v

- sửa thành node -v tương ứng
sed -i 's/yarn global add node-gyp/yarn global add node-gyp@10.3.1/' package.json

- Cài antlr4
apt install -y antlr4

- build maven
mvn -DskipTests clean package

- Giải nén và chạy
cd /root/start/openmetadata/openmetadata/openmetadata-dist/target
tar -xzf openmetadata-1.4.8.tar.gz

cd /root/start/openmetadata/openmetadata/openmetadata-dist/target/openmetadata-1.4.8
./bootstrap/bootstrap_storage.sh migrate-all
cd openmetadata-1.4.8
./bin/openmetadata-server-start.sh conf/openmetadata.yaml

- File cấu hình
/root/start/openmetadata/openmetadata/openmetadata-dist/target/openmetadata-1.4.8/conf/openmetadata.yaml



------------Flink------------
- Clone code:

mkdir -p ~/start/flink
cd ~/start/flink
git clone --branch release-1.18.1 --depth 1 https://github.com/apache/flink.git flink-1.18.1
cd ~/start/flink/flink-1.18.1


- Cai JDK 11 va Maven:
sudo apt update
sudo apt install -y openjdk-11-jdk maven
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
export PATH="$JAVA_HOME/bin:$PATH"
java -version
mvn -version


- Build Flink binary distribution:
mvn -DskipTests -Dfast -Drat.skip=true clean package

mvn \
  -DskipTests \
  -Dfast \
  -Drat.skip=true \
  -pl :flink-dist_2.12 \
  -am \
  install

- Run
./build-target/bin/start-cluster.sh

----------Polaris----------
Polaris

- Clone source
git clone \
  --branch apache-polaris-1.5.0 \
  --depth 1 \
  https://github.com/apache/polaris.git \
  polaris-1.5.0

- Cài jdk 21
sudo apt install -y openjdk-21-jdk
export JAVA_HOME=/usr/lib/jvm/java-21-openjdk-amd64
export PATH="$JAVA_HOME/bin:$PATH"

java -version

- Build
./gradlew assemble

- Run
./gradlew run \
  -Dpolaris.bootstrap.credentials=POLARIS,root,s3cr3t \
  -Dquarkus.http.host=0.0.0.0

----------Soda----------
Soda

- Clone 
git clone \
  --branch v4.17.0 \
  --depth 1 \
  https://github.com/sodadata/soda-core.git \
  soda-core-4.17.0

- Cài python 3.12 tạo môi trường 
sudo apt install -y python3.12 python3.12-venv python3-pip

python3.12 -m venv .venv
source .venv/bin/activate


python -m pip install --upgrade pip

- Run
pip install -e ./soda-core -e ./soda


----------MINIO----------
MINIO
- Clone
git clone \
  --branch RELEASE.2025-09-07T16-13-09Z \
  --depth 1 \
  https://github.com/minio/minio.git \
  minio-RELEASE.2025-09-07T16-13-09Z

- Cài go 1.24.2
sudo snap install go --classic --channel=1.24/stable
export PATH="/snap/bin:$PATH"

- Build
make build

- Run
mkdir -p /root/start/minio/data

export MINIO_ROOT_USER=minioadmin
export MINIO_ROOT_PASSWORD='MinioAdmin123!'

./minio server \
  /root/start/minio/data \
  --address 0.0.0.0:19000 \
  --console-address 0.0.0.0:19001
