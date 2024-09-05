# Supabase Kubernetes

This repository contains the charts to deploy a [Supabase](https://github.com/supabase/supabase) instance inside a Kubernetes cluster using Helm 3.

For any information regarding Supabase or this project, please [go to the official repository](https://github.com/supabase-community/supabase-kubernetes).


## Install (Example)
```
helm repo add supabase "https://tablecheck-labs.github.io/supabase-kubernetes/"
helm install supabase/supabase -f values.yaml
```

## License

[Apache 2.0 License.](./LICENSE)
