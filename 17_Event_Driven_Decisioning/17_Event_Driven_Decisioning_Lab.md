Event Driven Decision
=====================

**High available event-driven decisioning reference implementation.**

It’s a reference implementation for high available event-driven decisioning on Red Hat OpenShift Container Platform is available. You can customize this reference implementation to deploy Drools engine code that requires stateful processing, including rules developed with complex event processing, in an OpenShift environment. Doing this enables the decision engine to process complex event series with high availability.

**Goal.**

Install and test the **event-driven decisioning reference implementation**

**Prerequisite.**

- Having access to an OpenShift v4.2.x.

Enable AMQ Streams Operator
===========================

This activity is the only one that requires the cluster administration privileges (`cluster-admin`). Open the OpenShift web console and install the AMQ Streams Operator (v1.2):

-   On the side menu, select **Operators &gt; OperatorHub**

-   Select **AMQ Streams**

-   Leave all the default options and click **Subscribe**

Install the supporting Kafka cluster
====================================

The rest of the lab will be performed using `oc` command line.

The reference implementation can be retrieved in the file:`rhpam-7.5.0-reference-implementation.zip`

Unfortunately, there is a dependency problem. To workaround this, we’ll work with the upstream project.

1.  Clone the upstream repository:

        git clone --depth 1 git@github.com:kiegroup/openshift-drools-hacep.git
        cd openshift-drools-hacep

2.  Create `hacep` project or choose another name.

        oc new-project hacep

3.  Create the Kafka cluster named `my-cluster`

        cat << EOF | oc apply -f -
        apiVersion: kafka.strimzi.io/v1beta1
        kind: Kafka
        metadata:
          name: my-cluster
        spec:
          kafka:
            version: 2.2.1
            replicas: 3
            listeners:
              external:
                type: route
              plain: {}
            config:
              offsets.topic.replication.factor: 3
              transaction.state.log.replication.factor: 3
              transaction.state.log.min.isr: 2
              log.message.format.version: '2.2'
            storage:
              type: ephemeral
          zookeeper:
            replicas: 3
            storage:
              type: ephemeral
          entityOperator:
            topicOperator: {}
            userOperator: {}
        EOF

4.  Enter in the **hacep** project folder: `openshift-drools-hacep`

5.  Crete the Kafka topics to support the hacep execution:

        ls kafka-topics/*yaml |xargs -l1 oc apply -f

6.  Check that the Kafka cluster is ready:

        oc get pods
        NAME                                          READY   STATUS    RESTARTS   AGE
        my-cluster-entity-operator-584655864b-4nrwc   3/3     Running   0          5m28s
        my-cluster-Kafka-0                            2/2     Running   0          6m2s
        my-cluster-Kafka-1                            2/2     Running   0          6m2s
        my-cluster-Kafka-2                            2/2     Running   0          6m2s
        my-cluster-zookeeper-0                        2/2     Running   0          7m33s
        my-cluster-zookeeper-1                        2/2     Running   0          7m33s
        my-cluster-zookeeper-2                        2/2     Running   0          7m33s

Deploy the Rule Engine
======================

1.  Build all the projects business logic

        mvn clean install -DskipTests

2.  Create the Rule Engine image packaged as Spring Boot application

    1.  Switch to `springboot` folder

    2.  Edit `Dockerfile` to change the image name in `registry.access.redhat.com/ubi8/ubi-minimal:8.0`

    3.  Create the binary image

            oc new-build --binary --strategy=docker --name openshift-kie-springboot
            oc start-build openshift-kie-springboot --from-dir=. --follow

3.  Deploy the Rule Engine

    1.  The following steps are performed from the `springboot` folder

    2.  Create a service account with privileges to manage the ConfigMaps. A ConfigMap is used for the leader election.

            oc create -f kubernetes/service-account.yaml
            oc create -f kubernetes/role.yaml
            oc create -f kubernetes/role-binding.yaml

4.  Get the image name

        oc get is/openshift-kie-springboot -o template --template='{{range .status.tags}}{{range .items}}{{.dockerImageReference}}{{end}}{{end}}'

    1.  Open `kubernetes/deployment.yaml` and replace existing image URL with the result of the previous command.

    2.  Deploy the image

            oc apply -f kubernetes/deployment.yaml

Run the client sample (events injector)
=======================================

1.  Configure the SSL communication

    1.  enter in the client folder

            cd sample-hacep-project/sample-hacep-project-client

    2.  create the key store

            rm src/main/resources/keystore.jks
            keytool -genkeypair -keyalg RSA -keystore src/main/resources/keystore.jks

    3.  extract the Kafka cluster certification authority

            oc extract secret/my-cluster-cluster-ca-cert --keys=ca.crt --to=- > src/main/resources/ca.crt

    4.  add the Kafka CA to the client key store (in the following step we assume `password` as key store password, otherwise change it accordingly)

            keytool -import -trustcacerts -alias root -file src/main/resources/ca.crt -keystore src/main/resources/keystore.jks -storepass password -noprompt

2.  Configure the client

    1.  get the Kafka bootstrap endpoint with the following command

            oc get route/my-cluster-kafka-bootstrap

    2.  edit `src/main/resources/configuration.properties` to update the Kafka bootstrap server host (adding `:443` at the end) and the other details.

            ssl.keystore.location=src/main/resources/keystore.jks
            ssl.truststore.location=src/main/resources/keystore.jks
            ssl.keystore.password=password
            ssl.truststore.password=password
            bootstrap.servers=my-cluster-kafka-bootstrap-hacep.apps-crc.testing:443
            security.protocol=SSL

3.  Execute the client

        mvn exec:java -Dexec.mainClass="org.kie.hacep.sample.client.ClientProducerDemo"

Check the results on the Rule Engine
====================================

1.  Identify the Rule Engine leader

        oc get cm/default-leaders -o template --template='{{range $k,$v := .data}}{{if eq $k "leader.pod.null"}}{{printf "leader pod: %s\n" $v}}{{end}}{{end}}'

2.  Inspect the log of the leader pod. E.g. `oc logs -f openshift-kie-springboot-c8b9c6545-2p8x4`

3.  Check the presence of this information: `Price for RHT is<…> `

Troubleshooting
===============

-   The following warning on client side could be caused by an erroneous server host configuration, make sure that hostnames are resolved and the correct port is defined (443).

        WARN  o.a.Kafka.clients.NetworkClient - [Consumer clientId=consumer-1, groupId=drools] Connection to node -1 (my-cluster-Kafka-bootstrap-hacep.apps-crc.testing192.168.130.11:9094) could not be established. Broker may not be available.

