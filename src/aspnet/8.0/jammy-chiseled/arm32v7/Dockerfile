ARG REPO=mcr.microsoft.com/dotnet/runtime

# Installer image
FROM arm32v7/buildpack-deps:jammy-curl AS installer

# Retrieve ASP.NET Core
RUN aspnetcore_version=8.0.0-preview.2.23153.2 \
    && curl -fSL --output aspnetcore.tar.gz https://dotnetcli.azureedge.net/dotnet/aspnetcore/Runtime/$aspnetcore_version/aspnetcore-runtime-$aspnetcore_version-linux-arm.tar.gz \
    && aspnetcore_sha512='8d119b0cab829ddee58605a5902a6ce89066cf04d60a9ce3cd7be1be6bb4b42969ef98e073187a89ee0d948fe93361bcc7912840ea519cdae07cb32769c26e7f' \
    && echo "$aspnetcore_sha512  aspnetcore.tar.gz" | sha512sum -c - \
    && mkdir -p /dotnet \
    && tar -oxzf aspnetcore.tar.gz -C /dotnet ./shared/Microsoft.AspNetCore.App \
    && rm aspnetcore.tar.gz


# ASP.NET Core image
FROM $REPO:8.0.0-preview.2-jammy-chiseled-arm32v7

# ASP.NET Core version
ENV ASPNET_VERSION=8.0.0-preview.2.23153.2

COPY --from=installer ["/dotnet", "/usr/share/dotnet"]
