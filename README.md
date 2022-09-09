# Deploy Drupal with PostgreSQL to AWS ECS

### Docker Compose config (AWS ECS deployment):
```yaml
# Drupal with PostgreSQL in AWS ECS
#
# During initial Drupal setup:
# Database type: PostgreSQL
# Database name: postgres
# Database username: postgres
# Database password: [AWS Secret]
# ADVANCED OPTIONS; Database host: postgres

# x-aws-vpc: ECS VPC ID
# x-aws-cluster: ECS Cluster Name  
# Also EC2 network (Security Group ID) & AWS secret could be explicitly specified below

version: '3.1'

x-aws-vpc: "vpc-xxxxxxxxxxxxxxx"
x-aws-cluster: "your cluster name"

services:

  drupal:
    image: drupal:9.4-apache
    ports:
      - 80:80
    networks:
      - ec2_network
    volumes:
      - modules:/var/www/html/modules
      - profiles:/var/www/html/profiles
      - themes:/var/www/html/themes
      - sites:/var/www/html/sites
    restart: always

  postgres:
    image: postgres:10
    networks:
      - ec2_network
    volumes:
      - pgdata:/var/lib/postgresql/data
    secrets:
      - postgres_password
    environment:
       POSTGRES_PASSWORD_FILE: /run/secrets/postgres_password
    restart: always

networks:
  ec2_network:
    external: true
    name: "sg-xxxxxxxxxxxxxx"

volumes:
  modules:
  profiles:
  themes:
  sites:
  pgdata:

secrets:
  postgres_password:
    name: "arn:aws:secretsmanager:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
    external: true

x-aws-cloudformation:
  Resources:
    DrupalTCP80TargetGroup:
      Properties:
        HealthCheckPath: /
        Matcher:
          HttpCode: 200-499
```

### Deploy to AWS ECS:
```console
docker --context myecscontext compose -f aws-docker-compose.yaml up -d
```
Note: Running in detached mode "-d" is needed becase of the open issue: https://github.com/docker/compose-cli/issues/2086

---

### Docker Compose config (local deployment):

```yaml
# Drupal with PostgreSQL
#
# Access via "http://localhost:8080"
#   (or "http://$(docker-machine ip):8080" if using docker-machine)
#
# During initial Drupal setup,
# Database type: PostgreSQL
# Database name: postgres
# Database username: postgres
# Database password: example
# ADVANCED OPTIONS; Database host: postgres

version: '3.1'

services:

  drupal:
    image: drupal:9.4-apache
    ports:
      - 80:80
    volumes:
      - modules:/var/www/html/modules
      - profiles:/var/www/html/profiles
      - themes:/var/www/html/themes
      - sites:/var/www/html/sites
    restart: always

  postgres:
    image: postgres:10
    volumes:
      - pgdata:/var/lib/postgresql/data
    environment:
       POSTGRES_PASSWORD: example
    restart: always

volumes:
  modules:
  profiles:
  themes:
  sites:
  pgdata:
```

### Deploy locally:
```console
docker compose -f local-docker-compose.yaml up
```
Note: Running in detached mode "-d" is needed becase of the open issue: https://github.com/docker/compose-cli/issues/2086
