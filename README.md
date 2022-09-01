# Client CRs with Quarkus distribution

This repository contains examples with a temporary workaround for the missing Client CR in the [new Keycloak Operator](https://github.com/keycloak/keycloak/tree/main/operator).

## Motivation

To provide the best experience, the new Operator will use a new approach to manage Keycloak resources, such as Realms, Clients and Users. This approach will leverage the [new storage architecture](https://www.keycloak.org/2022/07/storage-map.html) and future immutability options, making the CRs the declarative single source of truth. In comparison to the [legacy Operator](https://github.com/keycloak/keycloak-operator), this will bring high robustness, reliability and predictability to the whole solution.

Before we would consider the new Operator ready for leveraging CRs, we expect completing several features including but not
limited to:

* File store (expected in Keycloak 20) to persist data in a file instead of DB.
* Read-only administration REST API, UI Console and other interfaces. This is required for the new immutability concept
  which will be used to ensure any data coming from the CRs (and subsequently from the file store) are read-only from
  all interfaces.

All of this is critical to proper CRs implementation, hence the new Operator is currently missing the CRDs for managing
Keycloak resources. The missing CRDs will be added once Keycloak has the necessary support for it, which is currently
expected in Keycloak 21.

## Using the Realm Import CR

One of the options to mitigate the current lack of CRs is the Realm Import CR. As the name suggests, this CR is meant only for declarative Realm creation (no updates or deletion is supported), very much like the [Realm CR in the legacy Operator](https://github.com/keycloak/keycloak-operator/blob/main/deploy/crds/keycloak.org_keycloakrealms_crd.yaml) which was used the same way. The main advantage and difference of the Realm Import CR are that it contains the full Realm representation (incl. all sub-resources like Clients, Users, etc.) unlike the old Realm CR which included only selected fields.

To learn more about the Realm Import CR, see the [documentation](https://www.keycloak.org/operator/realm-import).

## Using the legacy Operator for managing Clients

Another temporary workaround to the missing CRs is to use the legacy Operator in *tandem* with the new one. Due to the changes and enhancements in the Quarkus distribution of Keycloak, the legacy Operator is unable to deploy and manage its lifecycle and will remain to support only the legacy WildFly distribution. However, the Operator still can manage resources inside a running Quarkus distribution whose deployment is managed by the new Operator.

**NOTE:** The legacy Operator doesn't support creating Users in unmanaged external Realms (which in this example is created by the new Operator). Only Clients are supported.

### Example

1.  Deploy the new Operator.
    ```
    kubectl apply -f https://raw.githubusercontent.com/keycloak/keycloak-k8s-resources/19.0.1/kubernetes/keycloaks.k8s.keycloak.org-v1.yml
    kubectl apply -f https://raw.githubusercontent.com/keycloak/keycloak-k8s-resources/19.0.1/kubernetes/keycloakrealmimports.k8s.keycloak.org-v1.yml
    kubectl apply -f https://raw.githubusercontent.com/keycloak/keycloak-k8s-resources/19.0.1/kubernetes/kubernetes.yml
    ```
    Notice that we're using a hardcoded `19.0.1` version here. Feel free to use a newer version. See also the [installation guide](https://www.keycloak.org/operator/installation#_vanilla_kubernetes_installation).

2.  Clone this repository and check out this branch.

3.  Deploy the legacy Operator.
    ```
    kubectl apply -k legacy/deploy
    ```
    Notice that this is using a modified deployment to avoid name conflicts with the new Operator.

4.  Deploy Quarkus distribution of Keycloak using the new Operator.
    ```
    kubectl apply -f new/example-kc-deployment.yaml
    ```
    **WARNING:** For the simplicity of this example, the deployment is using a predefined `admin` username and password for the initial admin user.

    See also the [Basic Keycloak Deployment guide](https://www.keycloak.org/operator/basic-deployment).

5.  Import a Realm using the new Operator.
    ```
    kubectl apply -f new/example-realm.yaml
    ```

6.  Create Keycloak and Realm CRs for the legacy Operator to tell it where the external Keycloak lives.
    ```
    kubectl apply -f legacy/admin-credentials.yaml
    kubectl apply -f legacy/external-keycloak.yaml
    kubectl apply -f legacy/external-realm.yaml
    ```

7.  Deploy a Client using the legacy Operator to the `basic` Realm previously created by the new Operator.
    ```
    kubectl apply -f legacy/example-client.yaml
    ```
    This Client is fully managed by the legacy Operator, which means any updates to the CR will be synced to Keycloak. This is the main advantage over Realm Import CR (which can also contain Clients) from the new Operator.

## See also
* [The legacy Operator documentation](https://www.keycloak.org/docs/19.0.1/server_installation/index.html#_operator)
* [The new Operator documentation](https://www.keycloak.org/guides#operator)