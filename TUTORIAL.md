helm tutorial
-------------

Learning topics

* Cloud native apps - cockroachdb SQL database
* Intro to helm
* Best practices for consuming community helm charts
* Kubernetes storage - PersistentVolumes, PersistentVolumeClaims, StorageClasses
* StatefulSets
* Running commands in a container - oc rsh

cockroachdb overview
--------------------

Cockroach DB is a [cloud native app](https://www.cockroachlabs.com/product/cloud-native/) SQL database. While mostly a marketing term it translates roughly into an app that runs seamlessly on kubernetes without any modifications. In the case of cockroach it also has a helm chart and prometheus integration out of the box. Quick overview of [what a cloud native app is](https://www.ibm.com/cloud/learn/cloud-native)

intro to helm
-------------

* [Installation](https://github.com/helm/helm/releases/tag/v3.3.1) install the latest release of helm
* [Architecture](https://helm.sh/docs/topics/architecture/)
* [Charts](https://helm.sh/docs/topics/charts/)
* [Best Practices](https://helm.sh/docs/chart_best_practices/)


helm template
-------------

The helm template command provides a way to generate kubernetes manifests without actually applying the resources to the cluster. Before applying any helm chart to your cluster you should always template out the chart to a directory for inspection.

    helm template ./cockroachdb --output-dir=./output

After reading through the resulting manifest attempt to connect the templates with the values.yml file to understand how the final resources get generated.

This is a good time to review what a statefulset is and why you might want to use one for a database:

* [StatefulSet](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/) - Deployment that automatically provisions storage and gives containers a stable name. Distributed databases often require consistent stable host names in order to form proper quorums.


helm install
------------

Once you have confidence in the chart go ahead and install it

    helm upgrade --install cockroachdb ./cockroachdb -n my-namespace

If you want to override any of the values in values.yml you can add additonal files that override in order

    helm upgrade --install cockroachdb -f env/prod.yml -f env/cluster1.yml ./cockroachdb -n my-namespace

interacting with the app
------------------------

To interact with the cockaroach management console (port 8080) you should first expose the service with a route. You can either create a route manually or use the `oc expose` command. Navigate to the url once it's available. Also use the opportunity to review the prometheus metrics endpoint `/_status/vars`

    oc expose -h

You can interact with the database by logging into the pod and running the sql shell.

    oc rsh cockroachdb-0
    cockroach sql --insecure

Once you are logged into the sql command line try [creating a database and inserting some data](https://www.cockroachlabs.com/docs/stable/insert-data.html) Be sure to run a `select * from <table>` to verify the data is actually there.

persistent storage
------------------

To verify that persistent storage is functioning correctly scale the statefulset down to 0 and then back up again to 3. This will kill the running pods and force them to different nodes.

    oc edit statefulset cockroachdb

Log back into the database and run the `select * from <table>` statement again to show the storage is working

Use this opprutunity to also view the underlying storage

    oc get pv -o wide
    oc describe pv <name>
    oc get pvc -o wide
    oc describe pvc <name>
    oc get storageclass -o wide

Learn about these resources here:

* [StorageClass](https://kubernetes.io/docs/concepts/storage/storage-classes/) - Automatically provisions PVs
* [PersistentVolume](https://kubernetes.io/docs/concepts/storage/persistent-volumes/) Keeps track of provisined storage
* [PersistentVolumeClaim](https://kubernetes.io/docs/concepts/storage/persistent-volumes/) Storage that is attached and ready to be mounted to a pod.

wrapping up
-----------

Once you have finsished you can remove your helm installation and all associated resources.

    helm list
    helm delete <app name>



