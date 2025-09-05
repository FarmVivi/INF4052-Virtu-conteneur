FROM debian:13.0-slim

# Installer les outils pour se connecter à une base de données postgresql et clean les sources pour gagner de la place
RUN apt-get update \
	&& apt-get install -y --no-install-recommends postgresql-client \
	&& apt-get clean \
	&& rm -rf /var/lib/apt/lists/*

CMD [ "bash" ]
