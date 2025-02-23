# syntax=docker/dockerfile:1.3
#--------------------------------------------
# Dev Stage - Assembles and Builds Frontend
#--------------------------------------------
FROM --platform=$BUILDPLATFORM node:18-alpine as dev

ARG APP_SHELL_VERSION=next

RUN mkdir -p /app
WORKDIR /app


COPY packages/openmrs-esm-implementer-tools-app-6.1.2-lafia.tgz .
COPY packages/openmrs-esm-patient-registration-app-8.1.2-lafiamrs.tgz .
COPY packages/openmrs-esm-patient-search-app-8.2.1-lafiamrs.tgz .
COPY packages/openmrs-esm-primary-navigation-app-6.2.2-lafiamrs.tgz .
COPY packages/openmrs-esm-home-app-5.6.1-lafiamrs.tgz .
COPY packages/openmrs-esm-ward-app-8.2.1-lafiamrs.tgz .
COPY packages/openmrs-esm-service-queues-app-8.2.1-lafiamrs.tgz .
COPY packages/openmrs-esm-patient-list-management-app-8.2.1-lafiamrs.tgz .
COPY packages/openmrs-esm-appointments-app-8.2.1-lafiamrs.tgz .
COPY packages/openmrs-esm-laboratory-app-1.1.0-lafiamrs.tgz .
COPY packages/openmrs-esm-login-app-lafiahms-2.1.tgz .

COPY spa-assemble-config.json .
COPY spa-build-config.json .

RUN yarn add ./openmrs-esm-laboratory-app-1.1.0-lafiamrs.tgz
RUN yarn add ./openmrs-esm-implementer-tools-app-6.1.2-lafia.tgz
RUN yarn add ./openmrs-esm-patient-registration-app-8.1.2-lafiamrs.tgz
RUN yarn add ./openmrs-esm-patient-search-app-8.2.1-lafiamrs.tgz
RUN yarn add ./openmrs-esm-primary-navigation-app-6.2.2-lafiamrs.tgz
RUN yarn add ./openmrs-esm-home-app-5.6.1-lafiamrs.tgz
RUN yarn add ./openmrs-esm-ward-app-8.2.1-lafiamrs.tgz
RUN yarn add ./openmrs-esm-service-queues-app-8.2.1-lafiamrs.tgz
RUN yarn add ./openmrs-esm-patient-list-management-app-8.2.1-lafiamrs.tgz
RUN yarn add ./openmrs-esm-appointments-app-8.2.1-lafiamrs.tgz
RUN yarn add ./openmrs-esm-login-app-lafiahms-2.1.tgz

ARG CACHE_BUST
RUN npx --legacy-peer-deps openmrs@${APP_SHELL_VERSION:-next} assemble --manifest --mode config --config spa-assemble-config.json --target ./spa
RUN npx --legacy-peer-deps openmrs@${APP_SHELL_VERSION:-next} build --build-config spa-build-config.json --target ./spa
RUN if [ ! -f ./spa/index.html ]; then echo 'Build failed. Please check the logs above for details. This may have happened because of an update to a library that OpenMRS depends on.'; exit 1; fi

#--------------------------------------------
# Runtime Stage - Published Image
#--------------------------------------------
FROM nginx:1.25-alpine

RUN apk update && \
    apk upgrade && \
    # add more utils for sponge and envsubst
    apk add --no-cache moreutils

# clear any default files installed by nginx
RUN rm -rf /usr/share/nginx/html/*

COPY startup.sh /usr/local/bin/startup.sh
RUN chmod +x /usr/local/bin/startup.sh

COPY nginx.conf /etc/nginx/nginx.conf

COPY --from=dev /app/spa /usr/share/nginx/html
COPY config-core_demo.json /usr/share/nginx/html

#Replace logo
COPY resources/logo.png /usr/share/nginx/html
COPY resources/logo-white.png /usr/share/nginx/html
COPY resources/favicon.ico /usr/share/nginx/html

CMD ["/usr/local/bin/startup.sh"]
