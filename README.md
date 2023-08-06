# Target
Use the curl command be used in a Pod to retrieve all Pods in the `default` namespace.
## Service Account
In order to authenticate and authorize the interactions between Pods and the Kubernetes API server we first create a `Service Account` named: `curl-service-account`.
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: curl-service-account
  namespace: default
```
## Role
In order to get the permissions (RBAC rules) to access the specific Kubernetes API resources that the pod need, we first create `Role` named: `curl-role` allows service account associated with it to perform read and watch operations on pods within the default namespace.
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: curl-role
  namespace: default
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "list", "watch"]
```
## RoleBinding
In order to bind the permissions defined in the Role named "curl-role" to a specific ServiceAccount named "curl-service-account" within the default namespace, we create a `RoleBinding` named `curl-role-binding`.
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: curl-role-binding
  namespace: default
subjects:
  - kind: ServiceAccount
    name: curl-service-account
roleRef:
  kind: Role
  name: curl-role
  apiGroup: rbac.authorization.k8s.io
```
## Job
Finally, we create a one-time job to fulfill the target.
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: retrieve-job
  namespace: default
spec:
  completions: 1
  template:
    spec:
      serviceAccountName: curl-service-account
      containers:
        - name: curl-container
          image: curlimages/curl
          command: ["/bin/sh", "-c"]
          args:
          - |
            KUBERNETES_URL="https://$KUBERNETES_SERVICE_HOST:$KUBERNETES_SERVICE_PORT"
            TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
            curl -k -H "Authorization: Bearer $TOKEN" $KUBERNETES_URL/api/v1/namespaces/default/pods
      restartPolicy: Never
```
# Result
Once the Job completes, we can check the logs of the Pod created by the Job:
```shell
kubectl logs job/retrieve-job
```
## Output
<details>
<summary>Click to expand</summary>

```json
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
{
100 24217    0 24217    0     0  5291k      0 --:--:-- --:--:-- --:--:-- 5912k
  "kind": "PodList",
  "apiVersion": "v1",
  "metadata": {
    "resourceVersion": "27443"
  },
  "items": [
    {
      "metadata": {
        "name": "pod1",
        "namespace": "default",
        "uid": "39f42b61-6c43-40d2-aaf9-de585e904863",
        "resourceVersion": "909",
        "creationTimestamp": "2023-08-06T09:17:38Z",
        "annotations": {
          "kubectl.kubernetes.io/last-applied-configuration": "{\"apiVersion\":\"v1\",\"kind\":\"Pod\",\"metadata\":{\"annotations\":{},\"name\":\"pod1\",\"namespace\":\"default\"},\"spec\":{\"containers\":[{\"image\":\"nginx:latest\",\"name\":\"nginx\"}]}}\n"
        },
        "managedFields": [
          {
            "manager": "kubectl-client-side-apply",
            "operation": "Update",
            "apiVersion": "v1",
            "time": "2023-08-06T09:17:38Z",
            "fieldsType": "FieldsV1",
            "fieldsV1": {
              "f:metadata": {
                "f:annotations": {
                  ".": {},
                  "f:kubectl.kubernetes.io/last-applied-configuration": {}
                }
              },
              "f:spec": {
                "f:containers": {
                  "k:{\"name\":\"nginx\"}": {
                    ".": {},
                    "f:image": {},
                    "f:imagePullPolicy": {},
                    "f:name": {},
                    "f:resources": {},
                    "f:terminationMessagePath": {},
                    "f:terminationMessagePolicy": {}
                  }
                },
                "f:dnsPolicy": {},
                "f:enableServiceLinks": {},
                "f:restartPolicy": {},
                "f:schedulerName": {},
                "f:securityContext": {},
                "f:terminationGracePeriodSeconds": {}
              }
            }
          },
          {
            "manager": "kubelet",
            "operation": "Update",
            "apiVersion": "v1",
            "time": "2023-08-06T09:17:53Z",
            "fieldsType": "FieldsV1",
            "fieldsV1": {
              "f:status": {
                "f:conditions": {
                  "k:{\"type\":\"ContainersReady\"}": {
                    ".": {},
                    "f:lastProbeTime": {},
                    "f:lastTransitionTime": {},
                    "f:status": {},
                    "f:type": {}
                  },
                  "k:{\"type\":\"Initialized\"}": {
                    ".": {},
                    "f:lastProbeTime": {},
                    "f:lastTransitionTime": {},
                    "f:status": {},
                    "f:type": {}
                  },
                  "k:{\"type\":\"Ready\"}": {
                    ".": {},
                    "f:lastProbeTime": {},
                    "f:lastTransitionTime": {},
                    "f:status": {},
                    "f:type": {}
                  }
                },
                "f:containerStatuses": {},
                "f:hostIP": {},
                "f:phase": {},
                "f:podIP": {},
                "f:podIPs": {
                  ".": {},
                  "k:{\"ip\":\"10.244.0.8\"}": {
                    ".": {},
                    "f:ip": {}
                  }
                },
                "f:startTime": {}
              }
            },
            "subresource": "status"
          }
        ]
      },
      "spec": {
        "volumes": [
          {
            "name": "kube-api-access-jhxw9",
            "projected": {
              "sources": [
                {
                  "serviceAccountToken": {
                    "expirationSeconds": 3607,
                    "path": "token"
                  }
                },
                {
                  "configMap": {
                    "name": "kube-root-ca.crt",
                    "items": [
                      {
                        "key": "ca.crt",
                        "path": "ca.crt"
                      }
                    ]
                  }
                },
                {
                  "downwardAPI": {
                    "items": [
                      {
                        "path": "namespace",
                        "fieldRef": {
                          "apiVersion": "v1",
                          "fieldPath": "metadata.namespace"
                        }
                      }
                    ]
                  }
                }
              ],
              "defaultMode": 420
            }
          }
        ],
        "containers": [
          {
            "name": "nginx",
            "image": "nginx:latest",
            "resources": {},
            "volumeMounts": [
              {
                "name": "kube-api-access-jhxw9",
                "readOnly": true,
                "mountPath": "/var/run/secrets/kubernetes.io/serviceaccount"
              }
            ],
            "terminationMessagePath": "/dev/termination-log",
            "terminationMessagePolicy": "File",
            "imagePullPolicy": "Always"
          }
        ],
        "restartPolicy": "Always",
        "terminationGracePeriodSeconds": 30,
        "dnsPolicy": "ClusterFirst",
        "serviceAccountName": "default",
        "serviceAccount": "default",
        "nodeName": "minikube",
        "securityContext": {},
        "schedulerName": "default-scheduler",
        "tolerations": [
          {
            "key": "node.kubernetes.io/not-ready",
            "operator": "Exists",
            "effect": "NoExecute",
            "tolerationSeconds": 300
          },
          {
            "key": "node.kubernetes.io/unreachable",
            "operator": "Exists",
            "effect": "NoExecute",
            "tolerationSeconds": 300
          }
        ],
        "priority": 0,
        "enableServiceLinks": true,
        "preemptionPolicy": "PreemptLowerPriority"
      },
      "status": {
        "phase": "Running",
        "conditions": [
          {
            "type": "Initialized",
            "status": "True",
            "lastProbeTime": null,
            "lastTransitionTime": "2023-08-06T09:17:38Z"
          },
          {
            "type": "Ready",
            "status": "True",
            "lastProbeTime": null,
            "lastTransitionTime": "2023-08-06T09:17:53Z"
          },
          {
            "type": "ContainersReady",
            "status": "True",
            "lastProbeTime": null,
            "lastTransitionTime": "2023-08-06T09:17:53Z"
          },
          {
            "type": "PodScheduled",
            "status": "True",
            "lastProbeTime": null,
            "lastTransitionTime": "2023-08-06T09:17:38Z"
          }
        ],
        "hostIP": "192.168.49.2",
        "podIP": "10.244.0.8",
        "podIPs": [
          {
            "ip": "10.244.0.8"
          }
        ],
        "startTime": "2023-08-06T09:17:38Z",
        "containerStatuses": [
          {
            "name": "nginx",
            "state": {
              "running": {
                "startedAt": "2023-08-06T09:17:53Z"
              }
            },
            "lastState": {},
            "ready": true,
            "restartCount": 0,
            "image": "nginx:latest",
            "imageID": "docker-pullable://nginx@sha256:67f9a4f10d147a6e04629340e6493c9703300ca23a2f7f3aa56fe615d75d31ca",
            "containerID": "docker://53be7b24f8ce69d80b036e3a6c96f75c609c55854d043ee318c76f9cd5fa9043",
            "started": true
          }
        ],
        "qosClass": "BestEffort"
      }
    },
    {
      "metadata": {
        "name": "pod2",
        "namespace": "default",
        "uid": "86ae5ec9-f040-4e7d-a189-d569cc5f3896",
        "resourceVersion": "24866",
        "creationTimestamp": "2023-08-06T09:17:38Z",
        "annotations": {
          "kubectl.kubernetes.io/last-applied-configuration": "{\"apiVersion\":\"v1\",\"kind\":\"Pod\",\"metadata\":{\"annotations\":{},\"name\":\"pod2\",\"namespace\":\"default\"},\"spec\":{\"containers\":[{\"command\":[\"sleep\",\"3600\"],\"image\":\"busybox:latest\",\"name\":\"busybox\"}]}}\n"
        },
        "managedFields": [
          {
            "manager": "kubectl-client-side-apply",
            "operation": "Update",
            "apiVersion": "v1",
            "time": "2023-08-06T09:17:38Z",
            "fieldsType": "FieldsV1",
            "fieldsV1": {
              "f:metadata": {
                "f:annotations": {
                  ".": {},
                  "f:kubectl.kubernetes.io/last-applied-configuration": {}
                }
              },
              "f:spec": {
                "f:containers": {
                  "k:{\"name\":\"busybox\"}": {
                    ".": {},
                    "f:command": {},
                    "f:image": {},
                    "f:imagePullPolicy": {},
                    "f:name": {},
                    "f:resources": {},
                    "f:terminationMessagePath": {},
                    "f:terminationMessagePolicy": {}
                  }
                },
                "f:dnsPolicy": {},
                "f:enableServiceLinks": {},
                "f:restartPolicy": {},
                "f:schedulerName": {},
                "f:securityContext": {},
                "f:terminationGracePeriodSeconds": {}
              }
            }
          },
          {
            "manager": "kubelet",
            "operation": "Update",
            "apiVersion": "v1",
            "time": "2023-08-06T16:18:06Z",
            "fieldsType": "FieldsV1",
            "fieldsV1": {
              "f:status": {
                "f:conditions": {
                  "k:{\"type\":\"ContainersReady\"}": {
                    ".": {},
                    "f:lastProbeTime": {},
                    "f:lastTransitionTime": {},
                    "f:status": {},
                    "f:type": {}
                  },
                  "k:{\"type\":\"Initialized\"}": {
                    ".": {},
                    "f:lastProbeTime": {},
                    "f:lastTransitionTime": {},
                    "f:status": {},
                    "f:type": {}
                  },
                  "k:{\"type\":\"Ready\"}": {
                    ".": {},
                    "f:lastProbeTime": {},
                    "f:lastTransitionTime": {},
                    "f:status": {},
                    "f:type": {}
                  }
                },
                "f:containerStatuses": {},
                "f:hostIP": {},
                "f:phase": {},
                "f:podIP": {},
                "f:podIPs": {
                  ".": {},
                  "k:{\"ip\":\"10.244.0.7\"}": {
                    ".": {},
                    "f:ip": {}
                  }
                },
                "f:startTime": {}
              }
            },
            "subresource": "status"
          }
        ]
      },
      "spec": {
        "volumes": [
          {
            "name": "kube-api-access-qh6gw",
            "projected": {
              "sources": [
                {
                  "serviceAccountToken": {
                    "expirationSeconds": 3607,
                    "path": "token"
                  }
                },
                {
                  "configMap": {
                    "name": "kube-root-ca.crt",
                    "items": [
                      {
                        "key": "ca.crt",
                        "path": "ca.crt"
                      }
                    ]
                  }
                },
                {
                  "downwardAPI": {
                    "items": [
                      {
                        "path": "namespace",
                        "fieldRef": {
                          "apiVersion": "v1",
                          "fieldPath": "metadata.namespace"
                        }
                      }
                    ]
                  }
                }
              ],
              "defaultMode": 420
            }
          }
        ],
        "containers": [
          {
            "name": "busybox",
            "image": "busybox:latest",
            "command": [
              "sleep",
              "3600"
            ],
            "resources": {},
            "volumeMounts": [
              {
                "name": "kube-api-access-qh6gw",
                "readOnly": true,
                "mountPath": "/var/run/secrets/kubernetes.io/serviceaccount"
              }
            ],
            "terminationMessagePath": "/dev/termination-log",
            "terminationMessagePolicy": "File",
            "imagePullPolicy": "Always"
          }
        ],
        "restartPolicy": "Always",
        "terminationGracePeriodSeconds": 30,
        "dnsPolicy": "ClusterFirst",
        "serviceAccountName": "default",
        "serviceAccount": "default",
        "nodeName": "minikube",
        "securityContext": {},
        "schedulerName": "default-scheduler",
        "tolerations": [
          {
            "key": "node.kubernetes.io/not-ready",
            "operator": "Exists",
            "effect": "NoExecute",
            "tolerationSeconds": 300
          },
          {
            "key": "node.kubernetes.io/unreachable",
            "operator": "Exists",
            "effect": "NoExecute",
            "tolerationSeconds": 300
          }
        ],
        "priority": 0,
        "enableServiceLinks": true,
        "preemptionPolicy": "PreemptLowerPriority"
      },
      "status": {
        "phase": "Running",
        "conditions": [
          {
            "type": "Initialized",
            "status": "True",
            "lastProbeTime": null,
            "lastTransitionTime": "2023-08-06T09:17:38Z"
          },
          {
            "type": "Ready",
            "status": "True",
            "lastProbeTime": null,
            "lastTransitionTime": "2023-08-06T16:18:06Z"
          },
          {
            "type": "ContainersReady",
            "status": "True",
            "lastProbeTime": null,
            "lastTransitionTime": "2023-08-06T16:18:06Z"
          },
          {
            "type": "PodScheduled",
            "status": "True",
            "lastProbeTime": null,
            "lastTransitionTime": "2023-08-06T09:17:38Z"
          }
        ],
        "hostIP": "192.168.49.2",
        "podIP": "10.244.0.7",
        "podIPs": [
          {
            "ip": "10.244.0.7"
          }
        ],
        "startTime": "2023-08-06T09:17:38Z",
        "containerStatuses": [
          {
            "name": "busybox",
            "state": {
              "running": {
                "startedAt": "2023-08-06T16:18:05Z"
              }
            },
            "lastState": {
              "terminated": {
                "exitCode": 0,
                "reason": "Completed",
                "startedAt": "2023-08-06T15:18:03Z",
                "finishedAt": "2023-08-06T16:18:03Z",
                "containerID": "docker://af51e79c2d1aefb6add765f7f37c8a52ee35b43300536bb120ca52d23e522462"
              }
            },
            "ready": true,
            "restartCount": 7,
            "image": "busybox:latest",
            "imageID": "docker-pullable://busybox@sha256:3fbc632167424a6d997e74f52b878d7cc478225cffac6bc977eedfe51c7f4e79",
            "containerID": "docker://fa1763f480175ad5dd0e6a73ee3058e37c297a29f5ea546bd17502c1724d7242",
            "started": true
          }
        ],
        "qosClass": "BestEffort"
      }
    },
    {
      "metadata": {
        "name": "retrieve-job-4d5w4",
        "generateName": "retrieve-job-",
        "namespace": "default",
        "uid": "95d8cbbe-b865-4930-a7c3-ced0af80cdc6",
        "resourceVersion": "27434",
        "creationTimestamp": "2023-08-06T17:03:34Z",
        "labels": {
          "batch.kubernetes.io/controller-uid": "8f6e9348-79a9-425a-b3d2-d3e58782d6fb",
          "batch.kubernetes.io/job-name": "retrieve-job",
          "controller-uid": "8f6e9348-79a9-425a-b3d2-d3e58782d6fb",
          "job-name": "retrieve-job"
        },
        "ownerReferences": [
          {
            "apiVersion": "batch/v1",
            "kind": "Job",
            "name": "retrieve-job",
            "uid": "8f6e9348-79a9-425a-b3d2-d3e58782d6fb",
            "controller": true,
            "blockOwnerDeletion": true
          }
        ],
        "finalizers": [
          "batch.kubernetes.io/job-tracking"
        ],
        "managedFields": [
          {
            "manager": "kube-controller-manager",
            "operation": "Update",
            "apiVersion": "v1",
            "time": "2023-08-06T17:03:34Z",
            "fieldsType": "FieldsV1",
            "fieldsV1": {
              "f:metadata": {
                "f:finalizers": {
                  ".": {},
                  "v:\"batch.kubernetes.io/job-tracking\"": {}
                },
                "f:generateName": {},
                "f:labels": {
                  ".": {},
                  "f:batch.kubernetes.io/controller-uid": {},
                  "f:batch.kubernetes.io/job-name": {},
                  "f:controller-uid": {},
                  "f:job-name": {}
                },
                "f:ownerReferences": {
                  ".": {},
                  "k:{\"uid\":\"8f6e9348-79a9-425a-b3d2-d3e58782d6fb\"}": {}
                }
              },
              "f:spec": {
                "f:containers": {
                  "k:{\"name\":\"curl-container\"}": {
                    ".": {},
                    "f:args": {},
                    "f:command": {},
                    "f:image": {},
                    "f:imagePullPolicy": {},
                    "f:name": {},
                    "f:resources": {},
                    "f:terminationMessagePath": {},
                    "f:terminationMessagePolicy": {}
                  }
                },
                "f:dnsPolicy": {},
                "f:enableServiceLinks": {},
                "f:restartPolicy": {},
                "f:schedulerName": {},
                "f:securityContext": {},
                "f:serviceAccount": {},
                "f:serviceAccountName": {},
                "f:terminationGracePeriodSeconds": {}
              }
            }
          },
          {
            "manager": "kubelet",
            "operation": "Update",
            "apiVersion": "v1",
            "time": "2023-08-06T17:03:34Z",
            "fieldsType": "FieldsV1",
            "fieldsV1": {
              "f:status": {
                "f:conditions": {
                  "k:{\"type\":\"ContainersReady\"}": {
                    ".": {},
                    "f:lastProbeTime": {},
                    "f:lastTransitionTime": {},
                    "f:message": {},
                    "f:reason": {},
                    "f:status": {},
                    "f:type": {}
                  },
                  "k:{\"type\":\"Initialized\"}": {
                    ".": {},
                    "f:lastProbeTime": {},
                    "f:lastTransitionTime": {},
                    "f:status": {},
                    "f:type": {}
                  },
                  "k:{\"type\":\"Ready\"}": {
                    ".": {},
                    "f:lastProbeTime": {},
                    "f:lastTransitionTime": {},
                    "f:message": {},
                    "f:reason": {},
                    "f:status": {},
                    "f:type": {}
                  }
                },
                "f:containerStatuses": {},
                "f:hostIP": {},
                "f:startTime": {}
              }
            },
            "subresource": "status"
          }
        ]
      },
      "spec": {
        "volumes": [
          {
            "name": "kube-api-access-4pgkk",
            "projected": {
              "sources": [
                {
                  "serviceAccountToken": {
                    "expirationSeconds": 3607,
                    "path": "token"
                  }
                },
                {
                  "configMap": {
                    "name": "kube-root-ca.crt",
                    "items": [
                      {
                        "key": "ca.crt",
                        "path": "ca.crt"
                      }
                    ]
                  }
                },
                {
                  "downwardAPI": {
                    "items": [
                      {
                        "path": "namespace",
                        "fieldRef": {
                          "apiVersion": "v1",
                          "fieldPath": "metadata.namespace"
                        }
                      }
                    ]
                  }
                }
              ],
              "defaultMode": 420
            }
          }
        ],
        "containers": [
          {
            "name": "curl-container",
            "image": "curlimages/curl",
            "command": [
              "/bin/sh",
              "-c"
            ],
            "args": [
              "KUBERNETES_URL=\"https://$KUBERNETES_SERVICE_HOST:$KUBERNETES_SERVICE_PORT\"\nTOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)\ncurl -k -H \"Authorization: Bearer $TOKEN\" $KUBERNETES_URL/api/v1/namespaces/default/pods\n"
            ],
            "resources": {},
            "volumeMounts": [
              {
                "name": "kube-api-access-4pgkk",
                "readOnly": true,
                "mountPath": "/var/run/secrets/kubernetes.io/serviceaccount"
              }
            ],
            "terminationMessagePath": "/dev/termination-log",
            "terminationMessagePolicy": "File",
            "imagePullPolicy": "Always"
          }
        ],
        "restartPolicy": "Never",
        "terminationGracePeriodSeconds": 30,
        "dnsPolicy": "ClusterFirst",
        "serviceAccountName": "curl-service-account",
        "serviceAccount": "curl-service-account",
        "nodeName": "minikube",
        "securityContext": {},
        "schedulerName": "default-scheduler",
        "tolerations": [
          {
            "key": "node.kubernetes.io/not-ready",
            "operator": "Exists",
            "effect": "NoExecute",
            "tolerationSeconds": 300
          },
          {
            "key": "node.kubernetes.io/unreachable",
            "operator": "Exists",
            "effect": "NoExecute",
            "tolerationSeconds": 300
          }
        ],
        "priority": 0,
        "enableServiceLinks": true,
        "preemptionPolicy": "PreemptLowerPriority"
      },
      "status": {
        "phase": "Pending",
        "conditions": [
          {
            "type": "Initialized",
            "status": "True",
            "lastProbeTime": null,
            "lastTransitionTime": "2023-08-06T17:03:34Z"
          },
          {
            "type": "Ready",
            "status": "False",
            "lastProbeTime": null,
            "lastTransitionTime": "2023-08-06T17:03:34Z",
            "reason": "ContainersNotReady",
            "message": "containers with unready status: [curl-container]"
          },
          {
            "type": "ContainersReady",
            "status": "False",
            "lastProbeTime": null,
            "lastTransitionTime": "2023-08-06T17:03:34Z",
            "reason": "ContainersNotReady",
            "message": "containers with unready status: [curl-container]"
          },
          {
            "type": "PodScheduled",
            "status": "True",
            "lastProbeTime": null,
            "lastTransitionTime": "2023-08-06T17:03:34Z"
          }
        ],
        "hostIP": "192.168.49.2",
        "startTime": "2023-08-06T17:03:34Z",
        "containerStatuses": [
          {
            "name": "curl-container",
            "state": {
              "waiting": {
                "reason": "ContainerCreating"
              }
            },
            "lastState": {},
            "ready": false,
            "restartCount": 0,
            "image": "curlimages/curl",
            "imageID": "",
            "started": false
          }
        ],
        "qosClass": "BestEffort"
      }
    }
  ]
```

</details>