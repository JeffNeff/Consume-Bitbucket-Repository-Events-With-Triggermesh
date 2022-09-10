# Use Triggermesh to Ingest Bitbucket Events

There are tons of fun things we can pipe into Triggermesh through the [Webhook Source](https://docs.triggermesh.io/cloud/sources/webhook/).

A great use-case leveraging the Webhook Source, is the ingestion of repository update events from [Bitbucket](bitbucket.org).

During this post, we will be looking at how to set up a Triggermesh Webhook source to begin ingesting Bitbucket events into a simple event display.

### Deploy a Triggermesh Webhook Source.

* If you have Triggermesh installed in your cluster, you can deploy a Webhook Source and event viewer by applying the following yaml:

```yaml
apiVersion: sources.triggermesh.io/v1alpha1
kind: WebhookSource
metadata:
  name: sample
spec:
  eventType: com.example.mysample.event
  eventSource: instance-abc123

  eventExtensionAttributes:
    from:
    - path
    - queries

  basicAuthUsername: customuser
  basicAuthPassword:
    value: abc123secret

  sink:
    ref:
      apiVersion: serving.knative.dev/v1
      kind: Service
      name: event-display
---
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: event-display
spec:
  template:
    metadata:
    spec:
      containers:
        - image: >-
            docker.io/n3wscott/sockeye:v0.5.0@sha256:64c22fe8688a6bb2b44854a07b0a2e1ad021cd7ec52a377a6b135afed5e9f5d2

```

* If you do not have Triggermesh installed and would like to quickly set up a local development enviorment. Have a look at the [Triggermesh Quickstart](https://github.com/JeffNeff/Triggermesh-Quick-Start) repo!

* You can also deploy the above using `docker-compose` and expose it with something like [ngrok](https://ngrok.com/).
```yaml
services:
  webhook:
    platform: linux/amd64
    image: gcr.io/triggermesh/webhooksource-adapter
    environment:
      WEBHOOK_EVENT_TYPE: demo.event.type
      WEBHOOK_EVENT_SOURCE: demo.event.source
      K_SINK: "http://sockeye:8080"
    ports:
      - 8080:8080

  sockeye:
    platform: linux/amd64
    image:  docker.io/n3wscott/sockeye:v0.5.0@sha256:64c22fe8688a6bb2b44854a07b0a2e1ad021cd7ec52a377a6b135afed5e9f5d2
    ports:
      - 8080
```

```bash
ngrok http 8080
```

Then use the `ngrok` url moving forward :)

### Using the SAAS

* Navigate to cloud.triggermesh.io and login or sign up.

* Select `Bridges` from the left navigation.

* Click the `New Bridge` button, located in the top left corner, and then select "Create From Scratch".

* Click the `Source` box at the top of the bridge editor and then select `Webhook` from the list of available sources.

* Configure the source to your needs. For this example we will point the source to send events to the `default` broker with events of type `bitbucket.event` of source `bitbucket`.

* Select the `Targets` box at the bottom of the bridge, then select `Service` from the list of available targets.

* From the `Image Catalog` dropdown, select the `sockeye` service.

* Click save.


### Retrieve the Webhook URL

From kubectl:

```bash
kubectl get webhooksource <NAME> -o jsonpath='{.status.address.url}'
```

From the Triggermesh UI:

* Navigate to `Services` on the left

* Select the `WebhookSource` service

* View the URI

### Retrieve the Sockeye URL

Using the same steps as above, retrieve the Sockeye URL and naviagte to it in your browser. Keep this up to events that are sent later on.

### Configure Bitbucket Webhook(s)

* Navigate to your Bitbucket repository.

* Select `Repository Settings` from the left navigation.

* Select `Webhooks` from the left navigation, under `Workflow`.

* Click `Add webhook`.

* Provide a title and use the URL we retrieved from the previous step.

* Configure the webhook to send events for the events you want to receive.

* Click `Save`.


### Send and View Events

* Navigate to your Bitbucket repository.

* Incurr a change that will trigger on one of the configurables you set in the previous step.

* Navigate to the Sockeye URL in your browser and view the event.

I created a sample repository and a sample webhook that filters on `Push` events. After following the steps above, and pushing a small change to the repository, I was presented with the following event in the Sockeye event viewer:


```
application/json
id
2aa94e06-66bf-47b0-b1fb-009931eb7bfe
knativearrivaltime
2022-09-10T19:24:43.636565698Z
source
bitbucket
time
2022-09-10T19:24:43.628269545Z
type
bitbucket.events
{
  "push": {
    "changes": [
      {
        "old": {
          "name": "master",
          "target": {
            "type": "commit",
            "hash": "c9199d55ac9c91d7ce7e243a4ec124f40d163758",
            "date": "2022-09-10T19:21:12+00:00",
            "author": {
              "type": "author",
              "raw": "Jeff Neff <jeff@triggermesh.com>",
              "user": {
                "display_name": "Jeff Neff",
                "links": {
                  "self": {
                    "href": "https://api.bitbucket.org/2.0/users/%7B1c509971-1235-4f08-8a3f-aa55e3d6b60b%7D"
                  },
                  "avatar": {
                    "href": "https://secure.gravatar.com/avatar/5fd4c86ab7dd8b0c1e8fce7f1c3e788b?d=https%3A%2F%2Favatar-management--avatars.us-west-2.prod.public.atl-paas.net%2Finitials%2FJN-2.png"
                  },
                  "html": {
                    "href": "https://bitbucket.org/%7B1c509971-1235-4f08-8a3f-aa55e3d6b60b%7D/"
                  }
                },
                "type": "user",
                "uuid": "{1c509971-1235-4f08-8a3f-aa55e3d6b60b}",
                "account_id": "5e598c075a495e0c91a964bd",
                "nickname": "Jeff"
              }
            },
            "message": "README.md edited online with Bitbucket",
            "summary": {
              "type": "rendered",
              "raw": "README.md edited online with Bitbucket",
              "markup": "markdown",
              "html": "<p>README.md edited online with Bitbucket</p>"
            },
            "links": {
              "self": {
                "href": "https://api.bitbucket.org/2.0/repositories/tmjeff/demo/commit/c9199d55ac9c91d7ce7e243a4ec124f40d163758"
              },
              "html": {
                "href": "https://bitbucket.org/tmjeff/demo/commits/c9199d55ac9c91d7ce7e243a4ec124f40d163758"
              }
            },
            "parents": [
              {
                "type": "commit",
                "hash": "3bdcbee260422d6dda374e9ca0e01df3a3e92965",
                "links": {
                  "self": {
                    "href": "https://api.bitbucket.org/2.0/repositories/tmjeff/demo/commit/3bdcbee260422d6dda374e9ca0e01df3a3e92965"
                  },
                  "html": {
                    "href": "https://bitbucket.org/tmjeff/demo/commits/3bdcbee260422d6dda374e9ca0e01df3a3e92965"
                  }
                }
              }
            ],
            "rendered": {},
            "properties": {}
          },
          "links": {
            "self": {
              "href": "https://api.bitbucket.org/2.0/repositories/tmjeff/demo/refs/branches/master"
            },
            "commits": {
              "href": "https://api.bitbucket.org/2.0/repositories/tmjeff/demo/commits/master"
            },
            "html": {
              "href": "https://bitbucket.org/tmjeff/demo/branch/master"
            }
          },
          "type": "branch",
          "merge_strategies": [
            "merge_commit",
            "squash",
            "fast_forward"
          ],
          "default_merge_strategy": "merge_commit"
        },
        "new": {
          "name": "master",
          "target": {
            "type": "commit",
            "hash": "568e585abc62d7f22ad3d3a3dbbe6d9f5567d77d",
            "date": "2022-09-10T19:24:41+00:00",
            "author": {
              "type": "author",
              "raw": "Jeff Neff <jeff@triggermesh.com>",
              "user": {
                "display_name": "Jeff Neff",
                "links": {
                  "self": {
                    "href": "https://api.bitbucket.org/2.0/users/%7B1c509971-1235-4f08-8a3f-aa55e3d6b60b%7D"
                  },
                  "avatar": {
                    "href": "https://secure.gravatar.com/avatar/5fd4c86ab7dd8b0c1e8fce7f1c3e788b?d=https%3A%2F%2Favatar-management--avatars.us-west-2.prod.public.atl-paas.net%2Finitials%2FJN-2.png"
                  },
                  "html": {
                    "href": "https://bitbucket.org/%7B1c509971-1235-4f08-8a3f-aa55e3d6b60b%7D/"
                  }
                },
                "type": "user",
                "uuid": "{1c509971-1235-4f08-8a3f-aa55e3d6b60b}",
                "account_id": "5e598c075a495e0c91a964bd",
                "nickname": "Jeff"
              }
            },
            "message": "README.md edited online with Bitbucket",
            "summary": {
              "type": "rendered",
              "raw": "README.md edited online with Bitbucket",
              "markup": "markdown",
              "html": "<p>README.md edited online with Bitbucket</p>"
            },
            "links": {
              "self": {
                "href": "https://api.bitbucket.org/2.0/repositories/tmjeff/demo/commit/568e585abc62d7f22ad3d3a3dbbe6d9f5567d77d"
              },
              "html": {
                "href": "https://bitbucket.org/tmjeff/demo/commits/568e585abc62d7f22ad3d3a3dbbe6d9f5567d77d"
              }
            },
            "parents": [
              {
                "type": "commit",
                "hash": "c9199d55ac9c91d7ce7e243a4ec124f40d163758",
                "links": {
                  "self": {
                    "href": "https://api.bitbucket.org/2.0/repositories/tmjeff/demo/commit/c9199d55ac9c91d7ce7e243a4ec124f40d163758"
                  },
                  "html": {
                    "href": "https://bitbucket.org/tmjeff/demo/commits/c9199d55ac9c91d7ce7e243a4ec124f40d163758"
                  }
                }
              }
            ],
            "rendered": {},
            "properties": {}
          },
          "links": {
            "self": {
              "href": "https://api.bitbucket.org/2.0/repositories/tmjeff/demo/refs/branches/master"
            },
            "commits": {
              "href": "https://api.bitbucket.org/2.0/repositories/tmjeff/demo/commits/master"
            },
            "html": {
              "href": "https://bitbucket.org/tmjeff/demo/branch/master"
            }
          },
          "type": "branch",
          "merge_strategies": [
            "merge_commit",
            "squash",
            "fast_forward"
          ],
          "default_merge_strategy": "merge_commit"
        },
        "truncated": false,
        "created": false,
        "forced": false,
        "closed": false,
        "links": {
          "commits": {
            "href": "https://api.bitbucket.org/2.0/repositories/tmjeff/demo/commits?include=568e585abc62d7f22ad3d3a3dbbe6d9f5567d77d&exclude=c9199d55ac9c91d7ce7e243a4ec124f40d163758"
          },
          "diff": {
            "href": "https://api.bitbucket.org/2.0/repositories/tmjeff/demo/diff/568e585abc62d7f22ad3d3a3dbbe6d9f5567d77d..c9199d55ac9c91d7ce7e243a4ec124f40d163758"
          },
          "html": {
            "href": "https://bitbucket.org/tmjeff/demo/branches/compare/568e585abc62d7f22ad3d3a3dbbe6d9f5567d77d..c9199d55ac9c91d7ce7e243a4ec124f40d163758"
          }
        },
        "commits": [
          {
            "type": "commit",
            "hash": "568e585abc62d7f22ad3d3a3dbbe6d9f5567d77d",
            "date": "2022-09-10T19:24:41+00:00",
            "author": {
              "type": "author",
              "raw": "Jeff Neff <jeff@triggermesh.com>",
              "user": {
                "display_name": "Jeff Neff",
                "links": {
                  "self": {
                    "href": "https://api.bitbucket.org/2.0/users/%7B1c509971-1235-4f08-8a3f-aa55e3d6b60b%7D"
                  },
                  "avatar": {
                    "href": "https://secure.gravatar.com/avatar/5fd4c86ab7dd8b0c1e8fce7f1c3e788b?d=https%3A%2F%2Favatar-management--avatars.us-west-2.prod.public.atl-paas.net%2Finitials%2FJN-2.png"
                  },
                  "html": {
                    "href": "https://bitbucket.org/%7B1c509971-1235-4f08-8a3f-aa55e3d6b60b%7D/"
                  }
                },
                "type": "user",
                "uuid": "{1c509971-1235-4f08-8a3f-aa55e3d6b60b}",
                "account_id": "5e598c075a495e0c91a964bd",
                "nickname": "Jeff"
              }
            },
            "message": "README.md edited online with Bitbucket",
            "summary": {
              "type": "rendered",
              "raw": "README.md edited online with Bitbucket",
              "markup": "markdown",
              "html": "<p>README.md edited online with Bitbucket</p>"
            },
            "links": {
              "self": {
                "href": "https://api.bitbucket.org/2.0/repositories/tmjeff/demo/commit/568e585abc62d7f22ad3d3a3dbbe6d9f5567d77d"
              },
              "html": {
                "href": "https://bitbucket.org/tmjeff/demo/commits/568e585abc62d7f22ad3d3a3dbbe6d9f5567d77d"
              },
              "diff": {
                "href": "https://api.bitbucket.org/2.0/repositories/tmjeff/demo/diff/568e585abc62d7f22ad3d3a3dbbe6d9f5567d77d"
              },
              "approve": {
                "href": "https://api.bitbucket.org/2.0/repositories/tmjeff/demo/commit/568e585abc62d7f22ad3d3a3dbbe6d9f5567d77d/approve"
              },
              "comments": {
                "href": "https://api.bitbucket.org/2.0/repositories/tmjeff/demo/commit/568e585abc62d7f22ad3d3a3dbbe6d9f5567d77d/comments"
              },
              "statuses": {
                "href": "https://api.bitbucket.org/2.0/repositories/tmjeff/demo/commit/568e585abc62d7f22ad3d3a3dbbe6d9f5567d77d/statuses"
              },
              "patch": {
                "href": "https://api.bitbucket.org/2.0/repositories/tmjeff/demo/patch/568e585abc62d7f22ad3d3a3dbbe6d9f5567d77d"
              }
            },
            "parents": [
              {
                "type": "commit",
                "hash": "c9199d55ac9c91d7ce7e243a4ec124f40d163758",
                "links": {
                  "self": {
                    "href": "https://api.bitbucket.org/2.0/repositories/tmjeff/demo/commit/c9199d55ac9c91d7ce7e243a4ec124f40d163758"
                  },
                  "html": {
                    "href": "https://bitbucket.org/tmjeff/demo/commits/c9199d55ac9c91d7ce7e243a4ec124f40d163758"
                  }
                }
              }
            ],
            "rendered": {},
            "properties": {}
          }
        ]
      }
    ]
  },
  "repository": {
    "type": "repository",
    "full_name": "tmjeff/demo",
    "links": {
      "self": {
        "href": "https://api.bitbucket.org/2.0/repositories/tmjeff/demo"
      },
      "html": {
        "href": "https://bitbucket.org/tmjeff/demo"
      },
      "avatar": {
        "href": "https://bytebucket.org/ravatar/%7B13d76964-e79c-4ee2-8fec-8fe28e498b47%7D?ts=default"
      }
    },
    "name": "demo",
    "scm": "git",
    "website": null,
    "owner": {
      "display_name": "Jeff Neff",
      "links": {
        "self": {
          "href": "https://api.bitbucket.org/2.0/users/%7B1c509971-1235-4f08-8a3f-aa55e3d6b60b%7D"
        },
        "avatar": {
          "href": "https://secure.gravatar.com/avatar/5fd4c86ab7dd8b0c1e8fce7f1c3e788b?d=https%3A%2F%2Favatar-management--avatars.us-west-2.prod.public.atl-paas.net%2Finitials%2FJN-2.png"
        },
        "html": {
          "href": "https://bitbucket.org/%7B1c509971-1235-4f08-8a3f-aa55e3d6b60b%7D/"
        }
      },
      "type": "user",
      "uuid": "{1c509971-1235-4f08-8a3f-aa55e3d6b60b}",
      "account_id": "5e598c075a495e0c91a964bd",
      "nickname": "Jeff"
    },
    "workspace": {
      "type": "workspace",
      "uuid": "{1c509971-1235-4f08-8a3f-aa55e3d6b60b}",
      "name": "Jeff",
      "slug": "tmjeff",
      "links": {
        "avatar": {
          "href": "https://bitbucket.org/workspaces/tmjeff/avatar/?ts=1582926875"
        },
        "html": {
          "href": "https://bitbucket.org/tmjeff/"
        },
        "self": {
          "href": "https://api.bitbucket.org/2.0/workspaces/tmjeff"
        }
      }
    },
    "is_private": true,
    "project": {
      "type": "project",
      "key": "TM",
      "uuid": "{4f8ce6a2-cc4b-4f00-8e84-db284351e668}",
      "name": "tm",
      "links": {
        "self": {
          "href": "https://api.bitbucket.org/2.0/workspaces/tmjeff/projects/TM"
        },
        "html": {
          "href": "https://bitbucket.org/tmjeff/workspace/projects/TM"
        },
        "avatar": {
          "href": "https://bitbucket.org/account/user/tmjeff/projects/TM/avatar/32?ts=1662836104"
        }
      }
    },
    "uuid": "{13d76964-e79c-4ee2-8fec-8fe28e498b47}"
  },
  "actor": {
    "display_name": "Jeff Neff",
    "links": {
      "self": {
        "href": "https://api.bitbucket.org/2.0/users/%7B1c509971-1235-4f08-8a3f-aa55e3d6b60b%7D"
      },
      "avatar": {
        "href": "https://secure.gravatar.com/avatar/5fd4c86ab7dd8b0c1e8fce7f1c3e788b?d=https%3A%2F%2Favatar-management--avatars.us-west-2.prod.public.atl-paas.net%2Finitials%2FJN-2.png"
      },
      "html": {
        "href": "https://bitbucket.org/%7B1c509971-1235-4f08-8a3f-aa55e3d6b60b%7D/"
      }
    },
    "type": "user",
    "uuid": "{1c509971-1235-4f08-8a3f-aa55e3d6b60b}",
    "account_id": "5e598c075a495e0c91a964bd",
    "nickname": "Jeff"
  }
}
```
