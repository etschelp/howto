# Reproducible builds with npm

Some guidelines on how to get reproducible build results when working with node and npm.

## Version handling in package.json

- Only use exact versions, avoid using `~` or `^`
- Always check in `package-lock.json`, run `npm install` only after a version has changed, work with `npm ci` otherwise.

## Require or restrict a certain node/npm version

E.g. if you want to make sure that each developer uses a minimum version on a local machine.

package.json

```json
  "engineStrict": true,
  "engines": {
    "node": ">=18.13.0",
    "npm": ">=8.19.0"
  }
```

## Frontend maven plugin

This is helpful if you do not want everyone to install node/npm to be able to compile your frontend code, or if you do not have Docker available in your
build pipeline.

pom.xml
```xml
<build>
    <plugins>
        <plugin>
            <groupId>com.github.eirslett</groupId>
            <artifactId>frontend-maven-plugin</artifactId>
            <version>1.12.1</version>
            <configuration>
                <workingDirectory>frontend</workingDirectory>
            </configuration>
            <executions>
                <execution>
                    <id>install node and npm</id>
                    <phase>generate-resources</phase>
                    <goals>
                        <goal>install-node-and-npm</goal>
                    </goals>
                    <configuration>
                        <!-- See https://nodejs.org/en/download/ -->
                        <nodeVersion>v18.13.0</nodeVersion>
                        <npmVersion>8.19.3</npmVersion>
                    </configuration>
                </execution>
                <execution>
                    <id>npm install</id>
                    <phase>generate-resources</phase>
                    <goals>
                        <goal>npm</goal>
                    </goals>
                    <configuration>
                        <arguments>ci</arguments>
                    </configuration>
                </execution>
                <execution>
                    <id>npm run build</id>
                    <phase>generate-resources</phase>
                    <goals>
                        <goal>npm</goal>
                    </goals>
                    <configuration>
                        <arguments>run build</arguments>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

## Container build

### Build in Dockerfile

Dockerfile
```yml
# Frontend Build Step
FROM node:16-buster as FRONTEND
WORKDIR /frontend
COPY frontend .
RUN npm ci && npm run build

# Build runtime image
FROM caddy:2-alpine

COPY --from=FRONTEND /frontend/dist /usr/share/caddy
```

### GitHub action

.github/workflows/build.yml
```yml
      - name: Setup node and install deps
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          check-latest: true
      - run: npm --prefix frontend ci
      
      - name: Build frontend
        run: npm --prefix frontend run build
    
    # TODO bundle or image build
```

### Jenkins pipeline

See: https://www.jenkins.io/doc/book/pipeline/docker/

Jenkinsfile
```groovy
pipeline {
    agent {
        docker { image 'node:16-buster' }
    }
    stages {
        stage('build') {
            steps {
                sh 'npm ci && npm run build'
            }
        }
    }
}
```