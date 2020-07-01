---
title: Nice diagram about TLS, certificates and keys
date: 2020-01-20
tags: []
author: Maël Valais
---

```plain
 cert = (1) pub key, (2) information (3) signature of (1)+(2)
 self-signed cert = only (1) and (2)

   +--------------------------+ +-------------------------+
   |      CA CERTIFICATE      | |       ca cert key       |
   |+------------------------+| |+-----------------------+|
   ||      information       || ||      private key      ||
   |+------------------------+| |+-----------------------+|
   |+------------------------+| +-------------------------+
   ||       public key       ||               |
   ||                        ||               |
   |+------------------------+|               |
   +--------------------------+               |
                                              |
                                              |
   +--------------------------+               |
   |       CERTIFICATE        |               |
   |+------------------------+|               |
   ||1     information       ||               |
   |+------------------------+|               |
   |+------------------------+|               |
   ||2      public key       ||               |
   ||                        ||               |
   |+------------------------+|               |
   |+------------------------+|               |
   ||    signature of 1+2    ||<--------------+
   |+------------------------+|
   +--------------------------+
```

<!--
https://textik.com/#d85b4624473ca862
-->

## What's a "certificate" in the openssl jargon

For example, mitmproxy relies on a specific dir structure and filenames:

```sh
/Users/mvalais/.mitmproxy
├── mitmproxy-ca-cert.pem            # ca-cert = cert
└── mitmproxy-ca.pem                 # ca      = cert + key
```

That's the kind of directory structure that `--client-dir` would expect.

- https://ahmet.im/blog/kubectl-man-in-the-middle/

## Using `mitmproxy` + `kubectl` locally (Kind cluster) + client cert

```sh
# Let's use a separate kubeconfig file to avoid messing with ~/.kube/config.
export KUBECONFIG=~/.kube/kind-kind

# Now, let's create a cluster.
kind create cluster

# Let's see what the content of $KUECONFIG is:
cat $KUBECONFIG
```

Let's see this kubeconfig (`cat $KUBECONFIG`). We can see that it uses a
client cert as a way to authenticate to the apiserver.

```yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUN5RENDQWJDZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwcmRXSmwKY201bGRHVnpNQjRYRFRJd01EWXlPVEE1TURJeU4xb1hEVE13TURZeU56QTVNREl5TjFvd0ZURVRNQkVHQTFVRQpBeE1LYTNWaVpYSnVaWFJsY3pDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBS25qCmhmRzBvenZVb05jMXY1STkvYm13dFBqb2QvK0RyczF4TFZOcWgxQjhFcTY1S3lnQjBSbDdQcTJueUhyRVRnTmsKZ2VPQ2RzRURObGpKOE12SC9GbE9nRkUvS0dqK2hwN2hwc0dHRExReWFUOExUY25JN0FNL2t3KzY5R0hidlN3Nwo2V0Y1VERTMDU1RkRqdnRveGdHcERycmZQZTk5bXN5Zmk3aWtteDk5MmRyMHFQd0xxanJpZHNkWU52MUZqU1Y1CndkRlFISGxBS2hBcmlUWmpQMnhNL3poOENBOFhndjF0UUxVVk5IS1hrSG5UYWlkeFY1MkduaUVaQmd0d2tSK3oKQ3hQempVZFAxQ1JRYzU0YmxDYW9FQVdTc0NYTUVPREhTSnowSi9CWHJCU2JaeGZIakd0Y0k0bEhBUmx2aURNawp3U3lOLy9qdE9tbWhDT29BTzBzQ0F3RUFBYU1qTUNFd0RnWURWUjBQQVFIL0JBUURBZ0trTUE4R0ExVWRFd0VCCi93UUZNQU1CQWY4d0RRWUpLb1pJaHZjTkFRRUxCUUFEZ2dFQkFIMDBrYnZpaWNNT3IxdFJoTkQweVpGa3ZkT1AKUzFGOEZKK1BFd1o1WExUTVVyVG1yekVlZmgza21xWkxYUnlyK3c0Snk1a0grK1o3enBpdlp6Q3BGOEtwclJaWAp5N2Z6TkJOeWMrOHFKN3dCek0xZ21BdTRha3BlNVBYbkw3akZIcU9aUmNXTmZKcUpTci9BS05sK2c2SjAzV2pnCmJDc0NQNTNrZ1czSDZYMkRoS1JzZFRWQlc1UGVlek1YRVlON2VYbGpnWE4xcElMdEU1VE5oWVZIcTZaUDVCaDkKMmU2Y1U4bEFuNlhYQ0Ira2RsTkhudGp1cTlSL2lPQVJRaWpVSVlWSFdCT1NyWGp6TnhkTklEakRRd3lKUnViMApDVEhrcDg4eUlNaEV1Nk1OV1VCUU82ZnVqbURHbUJJbTlzdzJpSnI1UlArSUVUVnQyWFc0Q2dHT1cwVT0KLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=
    server: https://127.0.0.1:53744
  name: kind-helix
contexts:
- context:
    cluster: kind-helix
    user: kind-helix
  name: kind-helix
current-context: kind-helix
kind: Config
preferences: {}
users:
- name: kind-helix
  user:
    client-certificate-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUM4akNDQWRxZ0F3SUJBZ0lJYTZXc25HT0RGZW93RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXhNS2EzVmlaWEp1WlhSbGN6QWVGdzB5TURBMk1qa3dPVEF5TWpkYUZ3MHlNVEEyTWprd09UQXlNekJhTURReApGekFWQmdOVkJBb1REbk41YzNSbGJUcHRZWE4wWlhKek1Sa3dGd1lEVlFRREV4QnJkV0psY201bGRHVnpMV0ZrCmJXbHVNSUlCSWpBTkJna3Foa2lHOXcwQkFRRUZBQU9DQVE4QU1JSUJDZ0tDQVFFQXRkbkxsb0V1cnFoSmRrMGgKME5VcXpFSHhUbmVHS040QTNtWDBobmFLcE1TT3hISlBQcTB6V0t4WEVxNTZPTkdhSkhvS0VTRUM2Yjh2MGcwTQo5dXVGOXJyQ1VORkpXMmdHcFArRG96TEpVS3RtN2F0WFZRYXNVQ2tjbFRtQ003aHlqOGZTTlRDd2lxT3RNOUFvCnFVVTJYd1k0a2xhL0RuMkRBc1V1VmNiUTdITDA1N0tFbjZvVzlFVy9mQ2hMVE5OTWhmelpkNzFISGs3MFNOZDMKTG5jTXNjNmp2eS9kN2MwbjM4ek45SjVzWVZOTjNsS1YvOU5MT2FaTEQ0bSsyVU5XV1FVazRuci8rZjlaZk1IMwphVlFibDZSWEZwaW1jYWg4UjRJTmhRNkhYbmVEbUI2dUl3RGdjQnhhQUtoNFVPVlZwODB1Uy9pYUVLV1VSWE1PCkg4YXBZUUlEQVFBQm95Y3dKVEFPQmdOVkhROEJBZjhFQkFNQ0JhQXdFd1lEVlIwbEJBd3dDZ1lJS3dZQkJRVUgKQXdJd0RRWUpLb1pJaHZjTkFRRUxCUUFEZ2dFQkFJaWtmbis5TkJQc2FNSzNVMU5pWmtBaWxsM1pWVXJDdzNsWgpIeGRGbm9MZGxZYmtPeFVEN25EK3lrNXhZWW4yaDc3WUU4NlU5czJ3aitkdFJTR2Ezc3p4WVJQbkt5MDN3L1M4CkhqZzhwSkZOZFhHWlNPUWJDQTBmT1BONXl5MlpYRlVWd0JwZ3VTMC9nTkNUUkRPUDl3QmNjQXhiSGRmSjMxQzQKa2hCemF4QXI5UEliMVBzUlhqS2ZSRnkvcllwYWRBYVhhYmZMOFRvbTJ4cGpLVDYreGxoL3lJb2tZbVhxSnlXRgp1SGFmWG1qUEdyRDFoZUo2UnR5U0xRM0dKUXQxWmVPK3BaYlJlRjcxU0ZRSjRQdFhtTHhGMjZQL3dlV3F0NWFJClhTK0ZnanRYbzZqUG1mWWRweDZ3bWp3UHRaZ1pRdXhSL2tsejVvQnErNnFaYUZDWXg4RT0KLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=
    client-key-data: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFcEFJQkFBS0NBUUVBdGRuTGxvRXVycWhKZGswaDBOVXF6RUh4VG5lR0tONEEzbVgwaG5hS3BNU094SEpQClBxMHpXS3hYRXE1Nk9OR2FKSG9LRVNFQzZiOHYwZzBNOXV1RjlyckNVTkZKVzJnR3BQK0RvekxKVUt0bTdhdFgKVlFhc1VDa2NsVG1DTTdoeWo4ZlNOVEN3aXFPdE05QW9xVVUyWHdZNGtsYS9EbjJEQXNVdVZjYlE3SEwwNTdLRQpuNm9XOUVXL2ZDaExUTk5NaGZ6WmQ3MUhIazcwU05kM0xuY01zYzZqdnkvZDdjMG4zOHpOOUo1c1lWTk4zbEtWCi85TkxPYVpMRDRtKzJVTldXUVVrNG5yLytmOVpmTUgzYVZRYmw2UlhGcGltY2FoOFI0SU5oUTZIWG5lRG1CNnUKSXdEZ2NCeGFBS2g0VU9WVnA4MHVTL2lhRUtXVVJYTU9IOGFwWVFJREFRQUJBb0lCQVFDVkNJOXZJeVB0QkFKZwpyOG44NmhhUEc2UDFtTU1jandUTFAyZHRJNDF3aDU0eHBUVUl1czJQNkgzYjA1NWJIbnhqVkprWGZLUjBpTGxhClBsUFhzU0l6R00vVGlCSEVsYmFNVnRPOVZndml6dllsNWZ4R3RKZFhncm5vR2g5NDM3c1Qxc0dSMGZ0OVE3TFkKK2NtNUgvMzFWcFhhYUxsZjJNRWI3aG1STnNWV1lXWm9MUHJ2QUJPemVmUnpKU0RONHU0M3M0eVpnNHJoL3IzWQo3YUhYOS9CSHpJRk93WTRNL1BRQzdYaFhDMXBIeWNPY2lSWVhFTkpuMTdQN2NOSkdoMnJ4dnc2OVQ1QW9rdXY5Ckcxd2lUNmVrVmZuaVYrSlVnL1lwcTNSRGxNdVFETDRZdCtGNG1zVGJtN3NFZk5yVVMzWGZFMFdmcjB2Wk1YN2sKMnE3dXkxNEJBb0dCQU5jb1BxVHNBa3N5YzR6UWtmQmNzb0Y1ekdJM3NQTUFGTHRSMGF6S2x1Q1V3VGMzYVk5dgpiTE05OTBVK3VpVzdtRHF0VHpZMVNVSzZGNW0yMzJoSW5hTnBCTUVJcDZHQ0I3dis5WWZIQURVSkljaVdodmJhCmpIY0M5Qjl4SG5uUTBydU5aNUErdWVtRE9EZkRKUllIVUhrMEh3MlJpTE1qRFhVRmhBanJBUGJ4QW9HQkFOaGUKL3BHb2FWOWtUREtNOWZwTTZUQkZwMFVtb2lSb3crVG9TWjhWeVBmMkl1VlNpa0dtaXUydHU3MzZCZkJtVzNtQgpGWmFRc01rNkMydExkK1NBQ0lSbktQTUN4czYzR05WNkJ0aXRIRFlBaGNtRCtvU3VpaFRrd3l1WXlpQml4aFovCms1UDY3UHFKVkQ2YnhkNDV6YlFsS2VNaUVtMUhnQUVGSEF0UFBqbHhBb0dBYzNnaHhwankwakNOV3ZGRW9WN2UKWGlaanpnSmRjTXlHVTlHaFdiNlFJbzh5OHROR1Q3aFkrZ2t6ZjNJZXJNbDA5V2kxcmo0Q3gxRGdBWnJuWXl3MQpqZEY2djY1SmFLQkVUbHlTb1AvbjJJN0NGc2pTUGdFa2lXcUlZYWR2MTZoK3NERS9kMlp5bUNQWU0vVURIa05tCnFPV1VGTkFhTVNtS3UxYnVlV3JGNWNFQ2dZQnJvNTVySWVnQjM2aVVnVkdoU24rN1Z2dG15RmhqV29jUnFvbHQKamUzamhWeEl6eTRlaU5hV2RSWnY1U0R0UGs2RmZMVWJxVEY1ZWRuU2I4SGVOOStFMXJrbFk1MDVteGJNcEo4aApUY1U2RERxQ1RKamxSdHRFbDZXTVc3ODZLMGsyU2hORnk4LzJ0empreUtPLzhPdW5rZEZyd0RpQWl0QmdNWVdKCkRzdjYwUUtCZ1FDR2htYnZrODRaYm1sdERxYW9hOVR6YU5UT3FrN3dtb0tDby9FVEg5NEtNVVZMVmk3bHhpN1kKcXFQYUdiSm9aYkZGd0NvWW12Rk5jU21nYk9SNjdCVVBkdUEzMHo0VythV1pmQVkwZGpUR08yTC8zVVhoWHdHegpVUGpZTXZZQU5sdzFmZTVmZmk3UExJSGJvZXFhTmhhaDliRVJXdFNRbm95NnYzV2hlNTh6VkE9PQotLS0tLUVORCBSU0EgUFJJVkFURSBLRVktLS0tLQo=
```

Let's run mitmproxy using the above client cert and keys:

```sh
mitmproxy -p 9000 --ssl-insecure --set client_certs=<(kubectl config view --minify --flatten -o=go-template='{{(index (index .users 0).user "client-key-data")}}' | base64 -d && kubectl config view --minify --flatten -o=go-template='{{(index (index .users 0).user "client-certificate-data")}}' | base64 -d)
```

Now, let's make sure that we have some DNS alias to 127.0.0.1 to work
around [the Go proxy
limitation](https://maelvls.dev/go-ignores-proxy-localhost/) which skips
proxying for `127.0.0.1` and `localhost`:

```sh
grep "^127.0.0.1.*local$" /etc/hosts || sudo bash -c 'echo "127.0.0.1 local" >> /etc/hosts'
```

Finally you can run `kubectl`:

```sh
HTTPS_PROXY=:9000 kubectl get pods --kubeconfig=<(sed "s|127.0.0.1|local|" $KUBECONFIG) --insecure-skip-tls-verify
```

Let's see if it works for things like `kubectl exec` and `kubectl logs`:

```sh
% HTTPS_PROXY=:9000 kubectl --kubeconfig=<(sed "s|127.0.0.1|local|" $KUBECONFIG) --insecure-skip-tls-verify logs -n kube-system kindnet-6h8pm
I0701 11:11:22.156979       1 main.go:64] hostIP = 172.17.0.2
podIP = 172.17.0.2
I0701 11:11:33.239138       1 main.go:104] Failed to get nodes, retrying after error: Get https://10.96.0.1:443/api/v1/nodes: net/http: TLS handshake timeout
I0701 11:11:33.328698       1 main.go:150] handling current node
```