# Integration of Dex as Hydra Login Provider

## Introduction

This chart bootstraps an [ORY Hydra](https://www.ory.sh/docs/hydra/) "login and consent" application capable of performing Dex-based logins on a [Kyma](https://kyma-project.io) cluster.

ORY Hydra does not perform any user authentication by itself. This must be provided externally.
Hydra offers REST-based extension points to integrate with an external login provider.
A small application that runs along with Hydra server and takes part in Hydra's login and consent flows is required.
This chart provides such an application, it delegates login requests to the Dex running in a Kyma cluster.
In this way, we can use Hydra to issue Id Tokens for users authenticated by one of the available Dex authentication methods.


## Prerequisites
- Ability to install Kyma from sources, or a running Kyma installation with **ory** component installed.

## Installation

The installation assumes installation from Kyma sources. There is a way to install this chart on already running cluster, assuming that **ory** component is already installed. Look for additional information in _Note_ sections.

<br/>

1. Add new static client to Dex configuration in the **resources/dex/templates/dex-config-map.yaml** file.
   - Add this entry to the **staticClients** list:
       ```
       - id: hydra-integration
         name: 'Hydra Integration'
         redirectURIs:
         - 'https://oauth2-login-consent.{{ .Values.global.ingress.domainName }}/cb'
         secret: <secretValue>
     ```
     Replace `<secretValue>` with a strong password.

    _Note: Alternatively, you can change this in a running Kyma installation by modifying the respective Config Map resource and restarting Dex._

<br/>

2. Change hydra server configuration to point to a valid login-and-consent application.
   - Modify **resources/ory/charts/hydra/values.yaml** file and set the `loginConsent.name` attribute to the same value as defined in **this** chart: `oauth2-login-consent`

   _Note: Alternatively, you can change this in a running Kyma installation by modifying the values of_ **OAUTH2_CONSENT_URL** and **OAUTH2_LOGIN_URL** _environment variables of the Hydra server Deployment_ (`kubectl edit Deployment ory-hydra-oauth2 -n kyma-system`). _Ensure hydra server Pod is redeployed with new values._

<br/>

3. Install Kyma with **ory** component enabled.
   - Add the following to the components list in Installation CR instance used for Kyma Installation:
   ```
     - name: "ory"
       namespace: "kyma-system"
   ```

<br/>

4. Use Helm to install hydra-dex integration chart.
   - Run: `export DOMAIN_NAME=<domainName>`. Replace `<domainName>` with the proper domain name of Kyma ingress gateway for cluster installations or **kyma.local** for local installation. Consult Kyma installation documentation for details about getting domain name from the running Kyma installation.
   - Setup access to Tiller in order to use helm CLI. Consult Kyma installation documentation for details.
   - Run: `helm install . -n hydra-dex --namespace kyma-system --set loginConsent.domainName=${DOMAIN_NAME} --tls`

