# Bash

Bash scripts and notes

### Thanks to antoine guillemot (antoine.nokia.bogota@gmail.com) for helping me out deploying and making these scripts

``
#!/bin/bash

## NICE COLOR AND UTF8
GREENCHECK="\e[32m\e[1m\u2713\e[0m\e[39m"
GREENOK="\e[32mOK\e[39m"
REDCROSS="\e[91m\e[1m\u274C\e[0m\e[39m"
REDERROR="\e[91mERROR\e[39m"

if [[ "$1" == "test" ]]; then
        CONF="docker-compose-test.yml"
elif [[ "$1" == "prod" ]]; then
        CONF="docker-compose.yml"
else
        echo -e "[${REDERROR}] You must provide a full restart mode 'test' or 'prod'"
        echo "Usage: $0 test   -or-   $0 prod"
        exit
fi

if [[ ! -f $CONF ]]; then
        echo -e "[${REDERROR}] Can not find Compose file '${CONF}'. Please check that you are running this script inside a Docker directory."
fi

echo -n " - Checking that RabbitMQ is running..."

CONTAINER=$(docker ps -a --format "{{.Status}} {{.Names}}" |  grep -i ${PWD##*/} | grep rabbit)
if [[ $(echo -e "$CONTAINER" | wc -l) -ne 1 ]]; then
        echo -e "[${REDERROR}]\n\t > Looks like there is more than 1 rabbit running (or old snapshots still present. Check manually."
        exit
fi
if [[ $(echo -e "$CONTAINER" | cut -d' ' -f1 | grep Up | wc -l) -eq 1 ]]; then
        echo -e "[${GREENOK}]"
fi

CONTAINERNAME=$(echo -e "$CONTAINER" | rev | cut -d' ' -f1 | rev | cut -d'_' -f2- | rev | cut -d'_' -f2- | rev)
docker-compose -f $CONF exec $CONTAINERNAME /bin/bash -c 'rabbitmqctl list_queues'
``