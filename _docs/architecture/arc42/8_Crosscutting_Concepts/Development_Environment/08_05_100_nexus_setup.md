---
title: CI - Nexus Setup
tags: [arg42]
category: arc42
summary: "Nexus Setup"
---

# Übersicht

# Setup

`docker-compose.yml`
~~~yml
nexus:
    image: sonatype/nexus3
    volumes:
      - "nexus3-data:/sonatype-work"
    ports:
      - "8081:8081"
volumes:
  nexus3-data:
~~~

Hinweis: Volumes befinden sich unter `/var/lib/docker/volumes`.


# Konfiguration

## Repositories
![Nexus repositories](08_05_100_nexus/nexus_default_repositories.png "Nexus Repositories"){:height="300px" }

Details zum group Repository "maven-public" findet man in 
<https://help.sonatype.com/repomanager3/configuration/repository-management#RepositoryManagement-RepositoryGroup>. Dieses Repository 
wird als zentrales Maven Repository inklusive Mirror für Maven Central verwendet.

## User

Damit Jenkins Artefakte in Nexus deployen kann, legt man noch einen passenden User, z.B: "jenkins-nexus" an.

# Referenzen

* <https://hub.docker.com/r/sonatype/nexus3/>
* <https://help.sonatype.com/repomanager3/>  
* <https://help.sonatype.com/repomanager3/configuration/repository-management>  