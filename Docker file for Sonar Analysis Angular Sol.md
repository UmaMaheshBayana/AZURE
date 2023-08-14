Create ```sonar-scanner.proteries``` file in solution directory with below details.

```
sonar.organization=${SonarOrganizationName}
sonar.login=${SonarToken}
sonar.projectKey=${SonarProjectKey}
sonar.projectName=${SonarProjectName}
sonar.sources=/usr/src
sonar.exclusions= **/*.bin, **/node_modules/**/*, **/*.spec.ts
sonar.host.url=https://sonarcloud.io
sonar.qualityProfile=${Sonar-CSS}
sonar.qualityProfile=${Sonar-HTML}
sonar.qualityProfile=${Sonar-Javascript}
sonar.qualityProfile=${Sonar-Typescript}
sonar.branch.name=${SourceBarch}
sonar.test.inclusions=**/*.ts,**/*.html
#sonar.branch.target=develop
```

Now Create a Docker file and run it.

```
#stage 1
# Start a new image for SonarCloud analysis
FROM sonarsource/sonar-scanner-cli:latest AS sonarqube

# Set the working directory to /usr/src
WORKDIR /usr/src

# Copy the build files to Sonar source directory
COPY ./src /usr/src

# Copy Sonar scanner properties files from repo to socar conatiner
COPY ./sonar-scanner.properties /opt/sonar-scanner/conf/sonar-scanner.properties


# Run SonarCloud analysis
RUN sonar-scanner -Dsonar.verbose=true || (echo "SonarCloud analysis failed" && exit 1)

#stage 2
FROM node:latest as node
WORKDIR /app
COPY . .

RUN npm install --legacy-peer-deps
RUN npm run build-dev

#stage 3
FROM nginx:alpine
USER root

COPY --from=node /app/dist/$(outpufile) /usr/share/nginx/html

#Copy updated default.conf file with port 1025 to replace exeisting default.conf in the container.
COPY ./default.conf /etc/nginx/conf.d/default.conf

#RUN chgrp -R root /var/cache/nginx /var/run /var/log/nginx && \
#    chmod -R 770 /var/cache/nginx /var/run /var/log/nginx

EXPOSE 8080

```

