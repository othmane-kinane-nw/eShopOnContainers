#See https://aka.ms/containerfastmode to understand how Visual Studio uses this Dockerfile to build your images for faster debugging.

ARG NODE_IMAGE=node:12.0
FROM mcr.microsoft.com/dotnet/aspnet:6.0 AS base
WORKDIR /app
EXPOSE 80

FROM ${NODE_IMAGE} as node-build
WORKDIR /web/src
COPY Web/WebSPA/Client/package.json .
COPY Web/WebSPA/Client/package-lock.json .
RUN npm install -g npm@6
RUN npm install
COPY Web/WebSPA/Client .
RUN npm run build:prod

FROM mcr.microsoft.com/dotnet/sdk:6.0 AS prepare-restore-files
ENV PATH="${PATH}:/root/.dotnet/tools"
RUN dotnet tool install --global --no-cache dotnet-subset --version 0.3.2
WORKDIR /src
COPY . .
RUN dotnet subset restore "eShopOnContainers-ServicesAndWebApps.sln" --root-directory /src --output /src/restore_subset/

FROM mcr.microsoft.com/dotnet/sdk:6.0 AS build
WORKDIR /src
COPY --from=prepare-restore-files /src/restore_subset .
RUN dotnet restore "eShopOnContainers-ServicesAndWebApps.sln"

COPY . .
COPY --from=node-build /web/wwwroot /src/Web/WebSPA/wwwroot/
WORKDIR /src/Web/WebSPA
RUN dotnet publish --no-restore -c Release -o /app

FROM build AS publish

FROM base AS final
WORKDIR /app
COPY --from=publish /app .
ENTRYPOINT ["dotnet", "WebSPA.dll"]
