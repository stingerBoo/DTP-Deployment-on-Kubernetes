# Below image is from DTP:
FROM mysql:8

ENV MYSQL_ROOT_PASSWORD=root
ENV MYSQL_DATABASE=DTP
ENV MYSQL_USER=dtp
ENV MYSQL_PASSWORD=root
ENV ACCEPT_EULA=Y

# ADD init.sh /docker-entrypoint-initdb.d/init.sh
 ADD create.sql /docker-entrypoint-initdb.d
# CMD /bin/bash -c ./docker-entrypoint-initdb.d/create.sql
# RUN chmod a+r /docker-entrypoint-initdb.d/*
# COPY create.sql /docker-entrypoint-initdb.d/
# RUN mysql -u root -p root </docker-entrypoint-initdb.d/create.sql
# command: ["/bin/bash", "/init.sh"]