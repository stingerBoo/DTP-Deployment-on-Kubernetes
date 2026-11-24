# Below image is from DTP:
FROM parasoft/dtp-extension-designer:2026.2.0

WORKDIR /data

#RUN echo "=== BEFORE COPY: DATA/CONF ===" && ls -la /usr/local/parasoft/data/conf/ || true
#RUN echo "=== BEFORE COPY: DATA-TMP/CONF ===" && ls -la /usr/local/parasoft/data-tmp/conf/ || true

COPY --chown=parasoft:parasoft mysql-connector-java-8.0.28.jar /usr/local/parasoft/data-tmp/lib/thirdparty/

#RUN echo "=== AFTER COPY: DATA/CONF ===" && ls -la /usr/local/parasoft/data/conf/ || true
#RUN echo "=== AFTER COPY: DATA-TMP/CONF ===" && ls -la /usr/local/parasoft/data-tmp/conf/ || true

COPY --chown=parasoft:parasoft /_cert2026/. /usr/local/parasoft/dtp/tomcat/conf/_cert2026/
COPY --chown=parasoft:parasoft server.xml /usr/local/parasoft/dtp/tomcat/conf/

COPY --chown=parasoft:parasoft ssl.config.js /usr/local/parasoft/data-tmp/dtpservices/shared/

WORKDIR /usr/local/parasoft

# below commented RUN is to check if jar was copied.
#RUN dir /usr/local/parasoft/data/lib/thirdparty/