# nmstate operator 
```
oc project openshift-nmstate
oc edit ds nmstate-handler  <<< add --zap-log-level=debug
    spec:
      affinity: {}
      containers:
      - args:
        - --zap-log-level=debug
        - --zap-time-encoding=iso8601
        command:
        - manager
oc get pod -n openshift-nmstate
oc logs nmstate-handler-trvmw -f
```
