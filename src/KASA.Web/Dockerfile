# Build stage
FROM mcr.microsoft.com/dotnet/sdk:10.0 AS build
WORKDIR /src

COPY ["src/KASA.Web/KASA.Web.csproj", "src/KASA.Web/"]
COPY ["src/KASA.Application/KASA.Application.csproj", "src/KASA.Application/"]
COPY ["src/KASA.Domain/KASA.Domain.csproj", "src/KASA.Domain/"]

RUN dotnet restore "src/KASA.Web/KASA.Web.csproj"

COPY . .
WORKDIR "/src/src/KASA.Web"
RUN dotnet build "KASA.Web.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "KASA.Web.csproj" -c Release -o /app/publish /p:UseAppHost=false

# Runtime stage
FROM mcr.microsoft.com/dotnet/aspnet:10.0 AS final
WORKDIR /app

RUN apt-get update && apt-get install -y curl && rm -rf /var/lib/apt/lists/*

COPY --from=publish /app/publish .

EXPOSE 8081
ENV ASPNETCORE_URLS=http://+:8081
ENV ASPNETCORE_ENVIRONMENT=Production

HEALTHCHECK --interval=30s --timeout=10s --start-period=30s --retries=3 \
  CMD curl -f http://localhost:8081/health || exit 1

ENTRYPOINT ["dotnet", "KASA.Web.dll"]
