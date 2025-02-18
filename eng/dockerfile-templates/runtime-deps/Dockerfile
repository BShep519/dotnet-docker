{{
    set dotnetVersion to join(slice(split(PRODUCT_VERSION, "."), 0, 2), ".") ^
    set isAlpine to find(OS_ARCH_HYPHENATED, "Alpine") >= 0 ^
    set isDebian to find(OS_ARCH_HYPHENATED, "Debian") >= 0 ^
    set isUbuntu to find(OS_ARCH_HYPHENATED, "Ubuntu") >= 0 ^
    set isMariner to defined(match(OS_VERSION, "^cbl-mariner\d+\.\d+$")) ^
    set baseUrl to VARIABLES[cat("base-url|", dotnetVersion, "|", VARIABLES["branch"])] ^
    set isInternal to find(baseUrl, "msrc") >= 0 || find(baseUrl, "internal") >= 0 ^
    set baseImageRepo to when(isAlpine,
        cat(ARCH_VERSIONED, "/alpine"),
        when(isDebian,
            cat(ARCH_VERSIONED, "/debian"),
            when(isUbuntu,
                "ubuntu.azurecr.io/ubuntu",
                when(isMariner,
                    "mcr.microsoft.com/cbl-mariner/base/core",
                    "<NOT-IMPLEMENTED>")))) ^
    set baseImageTag to when(isAlpine || isMariner, OS_VERSION_NUMBER, OS_VERSION) ^
    set isRpmInstall to isMariner && dotnetVersion = "6.0" ^
    set isSingleStage to !(isRpmInstall && isInternal) ^
    set urlSuffix to when(isInternal, "$SAS_QUERY_STRING", "") ^
    set rpmFilename to "dotnet-runtime-deps.rpm" ^
    set utilPkgs to when(isMariner && dotnetVersion != "6.0" && dotnetVersion != "7.0", ["shadow-utils"], []) ^
    set username to "app" ^
    set uid to 64198 ^
    set gid to uid
}}{{
    if !isSingleStage:# Installer image
}}FROM {{baseImageRepo}}:{{baseImageTag}}{{if !isSingleStage: AS installer}}{{ if isInternal && isRpmInstall:

ARG SAS_QUERY_STRING

RUN {{InsertTemplate("Dockerfile.download-runtime-deps-pkg",
    [
        "url-suffix": urlSuffix,
        "filename": rpmFilename,
        "is-internal": isInternal
    ], "    ")}}}}
{{if isRpmInstall && isInternal:FROM {{baseImageRepo}}:{{baseImageTag}}
}}
RUN {{InsertTemplate("../Dockerfile.linux.install-deps")}}
{{ if isRpmInstall:
{{if isInternal:{{InsertTemplate("../Dockerfile.linux.copy-files",
[
    "files": [
        [
            "filename": rpmFilename
        ]
    ],
    "srcStage": "installer",
    "destination": ""
])}}
}}{{InsertTemplate("Dockerfile.install-runtime-deps-pkg",
    [
        "skip-download": isInternal
        "url-suffix": urlSuffix,
        "filename": rpmFilename
    ])}}
}}{{if dotnetVersion != "6.0" && dotnetVersion != "7.0":
# Create a non-root user and group
RUN {{InsertTemplate("Dockerfile.linux.non-root-user",
    [
        "name": username,
        "uid": uid,
        "gid": gid,
        "append-cmd": len(utilPkgs) > 0
    ])}}
}}
{{InsertTemplate("../Dockerfile.common-dotnet-envs")}}
