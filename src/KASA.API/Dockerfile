# Build stage
FROM mcr.microsoft.com/dotnet/sdk:10.0 AS build
WORKDIR /src

COPY ["src/KASA.API/KASA.API.csproj", "src/KASA.API/"]
COPY ["src/KASA.Application/KASA.Application.csproj", "src/KASA.Application/"]
COPY ["src/KASA.Domain/KASA.Domain.csproj", "src/KASA.Domain/"]
COPY ["src/KASA.Infrastructure/KASA.Infrastructure.csproj", "src/KASA.Infrastructure/"]

RUN dotnet restore "src/KASA.API/KASA.API.csproj"

COPY . .
WORKDIR "/src/src/KASA.API"
RUN dotnet build "KASA.API.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "KASA.API.csproj" -c Release -o /app/publish /p:UseAppHost=false

# Runtime stage
FROM mcr.microsoft.com/dotnet/aspnet:10.0 AS final
WORKDIR /app

RUN apt-get update && apt-get install -y curl && rm -rf /var/lib/apt/lists/*

COPY --from=publish /app/publish .

EXPOSE 8080
ENV ASPNETCORE_URLS=http://+:8080
ENV ASPNETCORE_ENVIRONMENT=Production

HEALTHCHECK --interval=30s --timeout=10s --start-period=30s --retries=3 \
  CMD curl -f http://localhost:8080/health || exit 1

ENTRYPOINT ["dotnet", "KASA.API.dll"]
