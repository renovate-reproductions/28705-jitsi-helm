# Renovate issue

This repository serves as an example of an issue with the kube-api datasource. When renovate scans the repository, it does not account for the templating used in Helm charts, resulting in it not scanning YAML. As a result, we want to update the API versions, but the templating of Helm charts hinders Renovate from detecting and correctly updating dependencies located in YAML files.

## Global Config

```
{
    "autodiscover": "false",
    "repositories": ["jitsi-helm"],
    "extends": [ "config:base",":disableDependencyDashboard"],
    "enabledManagers": ["kubernetes","helm","terraform"],
    "kubernetes": { "fileMatch": ["\\.ya?ml$"] },
    "packageRules": [
      {
        "enabled": true,
        "groupName": "Kubernetes Api",
        "matchDatasources": ["kubernetes-api"]
      },
      {
        "enabled": true,
        "groupName": "helm",
        "matchPackagePatterns": ["*"],
        "matchDatasources": ["helm"]
      },
      {
        "enabled": true,
        "groupName": "Terraform",
        "matchPackagePatterns": ["*"],
        "matchDatasources": ["terraform-module","terraform-provider"]
      },
      {
        "enabled": true,
        "groupName": "Kubernetes Api",
        "matchDatasources": ["kubernetes-api"]
      }
    ]
    }

```

## Environment variables

```
  RENOVATE_PLATFORM: "gitlab"
  RENOVATE_ENDPOINT: "https://gitlab/api/v4"
  RENOVATE_DRY_RUN: true
  RENOVATE_IGNORE_UNSTABLE: true
  RENOVATE_ONBOARDING: false
  RENOVATE_REQUIRE_CONFIG: "optional"
  RENOVATE_PRINT_CONFIG: false
  LOG_LEVEL: debug

```

## Current behavior

I see many logs with errors, example of such logs you can see below 

```

DEBUG: Failed to parse Kubernetes manifest. (repository=jitsi-meet, packageFile=templates/jigasi/configmap.yaml)
       "err": {
         "name": "YAMLException",
         "reason": "missed comma between flow collection entries",
         "mark": {
           "name": null,
           "buffer": "{{- if .Values.jigasi.enabled }}\napiVersion: v1\nkind: ConfigMap\nmetadata:\n  name: {{ include \"jitsi-meet.jigasi.fullname\" . }}\n  labels:\n    {{- include \"jitsi-meet.jigasi.labels\" . | nindent 4 }}\ndata:\n  JIGASI_BREWERY_MUC: '{{ .Values.jigasi.breweryMuc }}'\n  XMPP_SERVER: '{{ include \"jitsi-meet.xmpp.server\" . }}'\n  {{- range $key, $value := .Values.jigasi.extraEnvs }}\n  {{- if not (kindIs \"invalid\" $value) }}\n  {{ $key }}: {{ tpl $value $ | quote }}\n  {{- end }}\n  {{- end }}\n{{- end }}\n",
           "position": 2,
           "line": 0,
           "column": 2,
           "snippet": " 1 | {{- if .Values.jigasi.enabled }}\n-------^\n 2 | apiVersion: v1\n 3 | kind: ConfigMap"
         },
         "message": "missed comma between flow collection entries (1:3)\n\n 1 | {{- if .Values.jigasi.enabled }}\n-------^\n 2 | apiVersion: v1\n 3 | kind: ConfigMap",
         "stack": "YAMLException: missed comma between flow collection entries (1:3)\n\n 1 | {{- if .Values.jigasi.enabled }}\n-------^\n 2 | apiVersion: v1\n 3 | kind: ConfigMap\n    at generateError (/opt/containerbase/tools/renovate/37.61.4/node_modules/js-yaml/lib/loader.js:183:10)\n    at throwError (/opt/containerbase/tools/renovate/37.61.4/node_modules/js-yaml/lib/loader.js:187:9)\n    at readFlowCollection (/opt/containerbase/tools/renovate/37.61.4/node_modules/js-yaml/lib/loader.js:758:7)\n    at composeNode (/opt/containerbase/tools/renovate/37.61.4/node_modules/js-yaml/lib/loader.js:1442:11)\n    at readFlowCollection (/opt/containerbase/tools/renovate/37.61.4/node_modules/js-yaml/lib/loader.js:780:5)\n    at composeNode (/opt/containerbase/tools/renovate/37.61.4/node_modules/js-yaml/lib/loader.js:1442:11)\n    at readBlockMapping (/opt/containerbase/tools/renovate/37.61.4/node_modules/js-yaml/lib/loader.js:1104:12)\n    at composeNode (/opt/containerbase/tools/renovate/37.61.4/node_modules/js-yaml/lib/loader.js:1441:12)\n    at readDocument (/opt/containerbase/tools/renovate/37.61.4/node_modules/js-yaml/lib/loader.js:1625:3)\n    at loadDocuments (/opt/containerbase/tools/renovate/37.61.4/node_modules/js-yaml/lib/loader.js:1688:5)\n    at loadAll (/opt/containerbase/tools/renovate/37.61.4/node_modules/js-yaml/lib/loader.js:1701:19)\n    at extractApis (/opt/containerbase/tools/renovate/37.61.4/node_modules/renovate/lib/modules/manager/kubernetes/extract.ts:73:18)\n    at Object.extractPackageFile (/opt/containerbase/tools/renovate/37.61.4/node_modules/renovate/lib/modules/manager/kubernetes/extract.ts:34:8)\n    at extractPackageFile (/opt/containerbase/tools/renovate/37.61.4/node_modules/renovate/lib/modules/manager/index.ts:75:9)\n    at getManagerPackageFiles (/opt/containerbase/tools/renovate/37.61.4/node_modules/renovate/lib/workers/repository/extract/manager-files.ts:45:43)\n    at /opt/containerbase/tools/renovate/37.61.4/node_modules/renovate/lib/workers/repository/extract/index.ts:57:28\n    at async Promise.all (index 0)\n    at extractAllDependencies (/opt/containerbase/tools/renovate/37.61.4/node_modules/renovate/lib/workers/repository/extract/index.ts:54:26)\n    at extract (/opt/containerbase/tools/renovate/37.61.4/node_modules/renovate/lib/workers/repository/process/extract-update.ts:140:28)\n    at extractDependencies (/opt/containerbase/tools/renovate/37.61.4/node_modules/renovate/lib/workers/repository/process/index.ts:154:26)"
       }

```

## Expected behavior

The expected behavior is for Renovate to detect and correctly update the API versions in the YAML files located within Helm charts.

