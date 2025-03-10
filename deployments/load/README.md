# Helm Guide for Beekeeper

## Installing the Job

Deploy the Helm chart to install the Beekeeper job:

```bash
helm upgrade --install beekeeper-load-public ethersphere/beekeeper --namespace beekeeper -f ./beekeeper-load-public.yaml
```

## Uninstalling the Job

To uninstall the deployed Beekeeper job:

```bash
helm uninstall beekeeper-load-public --namespace=beekeeper
```

## Manually Start a Job from the Scheduled CronJob (Optional)

If you need to start a Beekeeper job immediately without waiting for the next scheduled run:

```bash
kubectl create job --from=cronjob/beekeeper-load-cronjob beekeeper-load-immediate --namespace=beekeeper
```

## Cleaning Up the Manual Job (Optional)

After the manual job has completed, you can clean it up to avoid clutter:

```bash
kubectl delete job beekeeper-load-immediate --namespace=beekeeper
```

## Notes

- The manual job does not interfere with the scheduled CronJob.
- Make sure to monitor the status of manual jobs using:

    ```bash
    kubectl get jobs --namespace=beekeeper
    ```

- To view the logs of the manual job, run:

    ```bash
    kubectl logs job/beekeeper-load-immediate --namespace=beekeeper
    ```
