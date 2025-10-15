
SHELL := /bin/bash

up:
	docker compose up -d --build

ready:
	for p in 7001 7002 7003 7004 7005 7006 7007; do 	  echo "----- $$p"; curl -s http://localhost:$$p/ready; echo; 	done

logs:
	docker compose logs -f otel-collector mongo-rs-setup mongodb loan-application-service

down:
	docker compose down

clean:
	docker compose down -v
