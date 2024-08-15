Create Jobs and CronJobs in Kubernetes.

## Step 1: Introduction

This guide is designed to provide you with a practical understanding of how to use Jobs and CronJobs in Kubernetes to manage and schedule tasks in your containerized applications.

Jobs and CronJobs are Kubernetes resources designed to manage and schedule tasks. They are particularly useful for running short-lived, batch, or scheduled tasks that need to complete successfully.

- `Jobs`: A Job is a Kubernetes resource that creates one or more Pods and ensures that a specified number of them successfully complete their designated task. Once the task is completed, the Job terminates. Jobs are ideal for running one-off, short-lived tasks or batch processing.
- `CronJobs`: A CronJob is an extension of the Job resource that allows you to schedule Jobs to run at specific intervals, such as every hour, daily, or on certain days of the week. CronJobs leverage the familiar cron syntax for specifying schedules, making it easy to automate repetitive tasks and maintenance work within your Kubernetes cluster.

## Step 2: Preparation

Please run the following command to install the k3s cluster:

curl -sfL https://get.k3s.io | INSTALL_K3S_VERSION=v1.28.5+k3s1 sh -

_Click to run on **node1**_

Verify the installation by running:

kubectl get nodes

_Click to run on **node1**_

## Step 3: Jobs

In this example, the Job creates a single Pod that runs the `busybox:1.34` image and executes a simple shell command. The `restartPolicy` is set to `Never`, and the `backoffLimit` is set to 4, which means Kubernetes will retry the Job up to 4 times if it fails.

Create a `job.yml`

job.yml 

`apiVersion: batch/v1 kind: Job metadata:   name: hello-job spec:   template:     spec:       containers:       - name: hello-job         image: busybox:1.34         args:         - /bin/sh         - -c         - date; echo Hello from the Kubernetes cluster       restartPolicy: Never   backoffLimit: 4`

_Click to create job.yml on **node1**_

To create the Job, run the following command:

kubectl apply -f job.yml

_Click to run on **node1**_

To check the status of the Job you just created, run:

kubectl get jobs

_Click to run on **node1**_

To see more details about the Job, such as the events and the status of the Pods it created, use the `describe` command. See that the Job completed in the Events (This may take some seconds to happen in this example).

kubectl describe job hello-job

_Click to run on **node1**_

See the job started a pod. After the execution the status is stated as `Completed`.

kubectl get pods -l=job-name=hello-job

_Click to run on **node1**_

After the creation, check if the job was successful. If it was successful you should see the message in the logs from the pod.

kubectl logs -l=job-name=hello-job

_Click to run on **node1**_

## Step 4: Running parallel Jobs and handling failures

Kubernetes Jobs can be configured to run multiple Pods in parallel to process tasks more quickly. In the Job spec, you can set the `completions` and `parallelism` fields to control the number of Pods that need to complete successfully and the number of Pods that can run concurrently.

parallel-job.yml 

`apiVersion: batch/v1 kind: Job metadata:   name: parallel-job spec:   completions: 5   parallelism: 2   template:     spec:       containers:       - name: worker         image: busybox         command: ["sh", "-c", "echo 'Processing task in parallel Job' && sleep 10"]       restartPolicy: OnFailure   backoffLimit: 6`

_Click to create parallel-job.yml on **node1**_

In this example, the Job will create 5 Pods in total, with a maximum of 2 running concurrently. The `restartPolicy` is set to `OnFailure`, so Pods will be restarted if they fail during execution.

Create the parallel Job and watch the pods being started

kubectl apply -f parallel-job.yml
watch kubectl get pods -l=job-name=parallel-job

_Click to run on **node1**_

Quit the `watch` command by pressing `Ctrl` and `C`

See that the job had 5 completions:

kubectl get jobs

_Click to run on **node1**_

## Step 5: Creating a simple CronJob

To create a simple CronJob, first, create a YAML file that defines the CronJob's specifications. Save the following example as `simple-cronjob.yml`:

simple-cronjob.yml 

`apiVersion: batch/v1 kind: CronJob metadata:   name: simple-cronjob spec:   schedule: "*/1 * * * *"   jobTemplate:     spec:       template:         spec:           containers:           - name: worker             image: busybox             command: ["sh", "-c", "echo 'Hello from the simple CronJob!' && date"]           restartPolicy: OnFailure`

_Click to create simple-cronjob.yml on **node1**_

In this example, the CronJob creates a new Job every minute, which runs a Pod with the busybox image and executes a simple shell command to print a message and the current date.

To create the CronJob, run the following command:

kubectl apply -f simple-cronjob.yml

_Click to run on **node1**_

Check the status of the cronjob

kubectl describe cronjob simple-cronjob

_Click to run on **node1**_

It is not executed yet, also check the pods until one is created:

watch kubectl get pods

_Click to run on **node1**_

CAUTION:

The job will be scheduled a minute after creation. So we have to wait a moment for the pods to appear.

Check the status of the cronjob again. See that it was executed. By default you should see the last 3 executions of the CronJob

kubectl describe cronjob simple-cronjob

_Click to run on **node1**_

### Scheduling and configuring CronJobs

The `schedule` field in the CronJob spec uses a cron-like expression to define the intervals at which new Jobs will be created. The format consists of five fields, representing minutes (0-59), hours (0-23), days of the month (1-31), months (1-12), and days of the week (0-7, where both 0 and 7 represent Sunday). The asterisk `*` represents any available value for the field.

In the previous example, the schedule was set to `"*/1 * * * *"`, which means "every minute". Some other examples of schedules:

- `"0 1 * * *"`: Run daily at 1:00 AM.
- `"0 */4 * * *"`: Run every 4 hours.
- `"0 0 1 * *"`: Run on the first day of every month.

### Macros

The following list represents all macros and the equivalent that is represented.

- `@yearly` (or `@annually`) Run once a year at midnight of 1 January.`0 0 1 1 *`
- `@monthly` Run once a month at midnight of the first day of the month. `0 0 1 * *`
- `@weekly` Run once a week at midnight on Sunday morning. `0 0 * * 0`
- `@daily` (or @midnight) Run once a day at midnight.`0 0 * * *`
- `@hourly` Run once an hour at the beginning of the hour. `0 * * * *`

### Additional settings

You can also configure additional settings for the CronJob, such as `concurrencyPolicy`, `startingDeadlineSeconds`, and `suspend`. For example:

INFO:

We will only tell you about this additional settings but will not use them in a hands-on part here. So you do not have to deploy anything. You may deploy the CronJob if you want to of course.

yml 

`apiVersion: batch/v1 kind: CronJob metadata:   name: configured-cronjob spec:   schedule: "*/1 * * * *"   concurrencyPolicy: Forbid   startingDeadlineSeconds: 30   suspend: false   jobTemplate:     spec:       template:         spec:           containers:           - name: worker             image: busybox             command: ["sh", "-c", "echo 'Hello from the configured CronJob!' && date"]           restartPolicy: OnFailure`

In this example, the `concurrencyPolicy` is set to `Forbid`, which means that if a new Job is scheduled while the previous Job is still running, the new Job will be skipped. The default is `Allow`. We can also terminate the old job by using `Replace`.

The `startingDeadlineSeconds` field is set to 30 seconds, which means that if the CronJob misses a scheduled execution for any reason, it has 30 seconds to start the Job; otherwise, the execution will be skipped.

The `suspend` field is set to `false`, which allows the CronJob to run; if set to `true`, the CronJob would be suspended, and no new Jobs would be created.

You can also define more options like timezones: See the [CronJob documentation](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/#time-zones)

## Step 6: Conclusion

In this hands-on tutorial, you've learned about Kubernetes `Jobs` and `CronJobs`, including their purpose, use cases, and key concepts. You've created simple and parallel Jobs, scheduled CronJobs, and learned how to monitor, manage, and troubleshoot them. Additionally, you've explored best practices and security considerations for working with Jobs and CronJobs.

By leveraging Kubernetes Jobs and CronJobs, you can efficiently run short-lived, batch, and scheduled tasks within your containerized applications, ensuring that your workloads are reliable, fault-tolerant, and automated. This knowledge will help you manage and optimize your Kubernetes cluster and improve the overall performance and maintainability of your applications.