# nmstate operator 
```
$ oc project openshift-nmstate
$ oc scale deployment nmstate-operator --replicas=0
$ oc edit ds nmstate-handler  <<< add --zap-log-level=debug
    spec:
      affinity: {}
      containers:
      - args:
        - --zap-log-level=debug
        - --zap-time-encoding=iso8601
        command:
        - manager
$ oc get pod -n openshift-nmstate

$ oc get pod -n openshift-nmstate
NAME                                      READY   STATUS      RESTARTS   AGE
nmstate-cert-manager-579ccd7f96-dpjkt     1/1     Running     0          4d15h
nmstate-console-plugin-574f6d9d8b-dplpw   0/1     Completed   0          4d15h
nmstate-console-plugin-574f6d9d8b-kx965   1/1     Running     0          30h
nmstate-handler-2nghb                     1/1     Running     0          23h
nmstate-handler-kt9zp                     1/1     Running     1          17h
nmstate-handler-xdlgc                     1/1     Running     0          40h
nmstate-webhook-58494f5597-dzp9k          1/1     Running     0          4d15h

$ oc logs nmstate-handler-trvmw -f
```
