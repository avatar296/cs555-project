DOCKER			?= docker
COMPOSE         ?= docker compose
PYTHON          ?= python3
PIP             ?= pip
VENV_DIR        ?= .venv

# Kafka / Topic
KAFKA_BROKER   ?= localhost:29092    # from host; use kafka:9092 inside containers

# Spark
SPARK_WORKER_COUNT ?= 2

# -------- Help --------
.PHONY: help
help:
	@echo "Targets:"
	@echo "  up              - start Kafka and Spark services"
	@echo "  down            - stop services and remove volumes"
	@echo "  logs            - tail compose logs"
	@echo "  kafka-topics    - list all Kafka topics"
	@echo "  kafka-describe  - describe topic details (default: trips.yellow)"
	@echo "  kafka-offsets   - show partition offsets for topic"
	@echo "  kafka-delete-topic - delete a topic"
	@echo "  venv         	 - create venv"
	@echo "  install         - create venv and install dependencies"
	@echo "  clean           - remove venv and caches"
	@echo "  clean-checkpoint - remove Spark checkpoint and state data"


# -------- Compose lifecycle --------
.PHONY: up down logs
up:
	$(COMPOSE) up -d --scale spark-worker=$(SPARK_WORKER_COUNT)

down:
	$(COMPOSE) down -v

logs:
	$(COMPOSE) logs -f --tail=200

# -------- Kafka utilities --------
.PHONY: kafka-topics kafka-describe kafka-offsets kafka-delete-topic

kafka-topics:
	$(DOCKER) exec kafka /opt/kafka/bin/kafka-topics.sh \
		--bootstrap-server localhost:9092 --list

kafka-describe:
	$(DOCKER) exec kafka /opt/kafka/bin/kafka-topics.sh \
		--bootstrap-server localhost:9092 \
		--describe --topic $(TOPIC)

kafka-offsets:
	@echo "Getting consumer groups and offsets..."
	@$(DOCKER) exec kafka /opt/kafka/bin/kafka-consumer-groups.sh \
		--bootstrap-server localhost:9092 \
		--all-groups \
		--describe || echo "No active consumer groups found"

kafka-delete-topic:
	$(DOCKER) exec kafka /opt/kafka/bin/kafka-topics.sh \
		--bootstrap-server localhost:9092 \
		--delete --topic $(TOPIC)

# -------- Python env (producer) --------
.PHONY: venv install
venv:
	$(PYTHON) -m venv $(VENV_DIR)
	@echo "Run: source $(VENV_DIR)/bin/activate"

install: venv
	. $(VENV_DIR)/bin/activate && \
	$(PIP) install --upgrade pip && \
	$(PIP) install -r requirements.txt


# -------- Spark streaming --------
.PHONY: spark-ui

spark-ui:
	@echo "Opening Spark Master UI..."
	@open http://localhost:8081 2>/dev/null || xdg-open http://localhost:8081 2>/dev/null || echo "Visit http://localhost:8081"


# -------- Clean up --------
.PHONY: clean

clean:
	rm -rf $(VENV_DIR)
	find . -type d -name "__pycache__" -exec rm -rf {} + 2>/dev/null || true
	find . -type f -name "*.pyc" -delete 2>/dev/null || true
