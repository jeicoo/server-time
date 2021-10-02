# server-time

Live Application: https://server-time.azurewebsites.net/

One-page application that displays the server time and its converted value into Asia/Manila timezone.

The application as a whole is composed of 3 components:

1. [backend-clock](https://github.com/jeicoo/backend-clock) - the `backend` component which is a spring boot application that servers 2 RESTful endpoint
2. [frontend-clock](https://github.com/jeicoo/frontend-clock) - the `frontend` component which is an Angluar application. being served via a static web server
3. [gateway-clock](https://github.com/jeicoo/server-time/tree/gateway-clock)(this repo) - a layer 7 load balancer(nginx) that acts as a gateway, routes the traffic to the frontend and backend servers. Being a gateway, this is the entrypoint of the incoming traffic, and connects the frontend and backend using a single `host/domain:port`

## Designing the application

- Somehow, the two applications, namely the backend and the frontend needs to communicate via the same `Host` header to be able to get past the CORS restriction of the browser. My solution is to handle the traffic in a single container, and route the requests based on their path. In this way, the backend and the frontend will be accessible via the same `Host`/domain.

- I opted to call the 3rd party API using the backend. This will make sure the API key used in requests to the API server will not be exposed in the browser. Having the 3rd party API call in the frontend side is not that secure. You will risk your API key being stolen as the API key will reside in the user-agent/browser and they will be accessible from the client.

## High Level Architecture




## Designing CI/CD

In order to design a CI process/pipeline for your application, one must consider the tools involved in a project - language, build tools, packaging, test automation, etc. For this assignment, I decided to use Azure's `App Service` product in which you can deploy container based application not needing to worry about the infrastracture of your application. All you need is a docker image and then you can make your application UP and available. With that in mind, we can say that the final package that our CI process needs to provide will be a docker image.

## CI Process

Considering all of those mentioned above, the ideal CI process for the backend and frontend projects will start with a simple branching strategy.

`release/*` - a release branch signify a working production-ready state of the codebase, every commit to this branch should trigger a pipeline with stages to `build the application`, `run automated unit tests`, `publish test results/code coverage`, `build final package - docker image`, `push docker image to a repository`

    For the backend project, the `building the application` stage should make use of `maven`, compiling the application and producing a `*.jar` file. The `run automated unit tests` should use `maven` as well to be able to orchestrate the whole testing suite. After running the test suite, there should be output files, the `publish test results/code coverage` stage should be able to publish those files in a way that I can be easily viewed and accessible within the pipeline run. For the `build final package step`, a simple docker build command should do. And for the last stage which is `push docker image to a repository`, the built image should then be pushed into a private/public image repository.

    For the frontend project, the whole CI process should be identical to those of the backend project, the only difference are the tools being used. For building the application, `npm` will should be used instead of maven and for running the tests, `karma` is the command to use to be able to run the test suite. of course, the pipeline should be configured to handle the test results. The rest of the stages should be the same as they both use docker to be able to produce a docker image.

    Should there be an error encountered in one of the stage, the whole pipeline should fail. The pipeline result should reflect in the repository of the project.


`task/*, or other branches that are not release/*` - these branches are for development purposes. The CI process for these branches should be only up to publishing test results. Commits to these branches are not candidates for deployment yet. The reason we have pipeline for these branches is that the pipeline should do the test for you. The pipeline will boost confidence in the commits that are being made as the changes will underogo compilation and tests. The details for frontend and backend projects CI process should be the same as above, after all, the implementation for the stages are exactly the same. Is only based on the branch name that a certain stage will run.

## CD Process

Since the target environment for this project is the Azure App Service. And since I am using a Free Tier Azure App Service Plan, I only have one environment at my disposal and the environment is a live one. The ideal CD process/setup for this project should contain a single stage which will deploy the images in my azure app service. The stage should make use of azure cli to be able to update my app service images. Once the Azure App Service got the request. Azure will update the app service with the desired image then the changes will be reflected in the live application. The stage can be used for both frontend and backend project as the deployment process should be the same.
