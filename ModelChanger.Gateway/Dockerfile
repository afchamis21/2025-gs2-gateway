FROM mcr.microsoft.com/dotnet/sdk:9.0 AS build
WORKDIR /src

ARG PROJECT_PATH

COPY $PROJECT_PATH .
RUN dotnet restore

COPY . .

RUN dotnet publish $PROJECT_PATH -c Release -o /app/publish /p:UseAppHost=false

FROM mcr.microsoft.com/dotnet/aspnet:9.0 AS final
WORKDIR /app
COPY --from=build /app/publish .

ENTRYPOINT ["dotnet", "ModelChanger.Gateway.dll"]