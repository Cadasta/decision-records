# 0006: Asynchronous Import and Export

- **Author:** Adrienne Platner
- **Status:** Submitted, 2017-01-26

## Context

The Cadasta platform currently relies on synchronous processes embedded within the platform for all imports and exports. This includes batch imports of project data, individual project resource uploads, and batch exports of project data.

These are resource-intensive, blocking processes that require the user to wait until completion before proceeding in the UI. The user experience includes noticeable delays and, for batch operations, timeouts when the process exceeds the maximum connection duration. We have already run into these problems under the existing system. When multiplying this effect for concurrent users, overall degraded platform performance will result and the system will not scale. The overhead in terms of both system resources and elapsed time for the user also precludes us from introducing security measures into the process, such as scanning uploaded files for malicious content. This exposes a potential attack vector; generating automated concurrent uploads of large import files would cripple the platform server, whether or not the files contained malicious content.

Our goals are:

1) Accommodate the batch import of large project data sets;
2) Implement security scanning of all files introduced to the system by platform users;
3) Improve the user experience by reducing and eliminating delays due to blocking processes;
4) Prevent negative performance impact to the platform during import and export;
5) Create extensibility for future features by decoupling these processes from the platform.

## Recommendation

The existing batch import process will be replaced with the following workflow:

1) User uploads file via platform UI directly to temporary S3 bucket;
    1a) If file upload fails, the user receives a notification of the error;
    1b) Optionally, on successful upload, platform code logs the import request details (project ID, requesting user, file name/path, datestamp, etc) to a table in the platform database;
2) An event notification containing the upload metadata is triggered by S3 to an SQS queue;
3) A scheduled process running on a worker node uses long polling to check the SQS queue and retrieve new messages;
4) The message is parsed for file data including full path;
5) The uploaded file is scanned for malicious content using clamAV;
    5a) If the file passes, it is processed and moved to its permanent S3 bucket;
    5b) If the file fails, it is moved to a quarantine S3 bucket and platform admins are alerted;
    5c) Optionally, status of the import request is written to the platform database;
6) The user is notified of the success or failure of the import, and either linked to their newly populated project content or advised of next steps.

Notes for this workflow: 
    File processing as referenced in step 4 would initially include importing the project data to the platform database, but may also extend to other operations for resources such as image thumbnail creation, image analysis for content (i.e. using AWS's built-in image processing feature to analyze image content and apply tags to denote the image content; building, tree, river, male person, female person, document, etc).

    For the import of individual resources, we can utilize a simplified version of the same workflow that still includes clamAV scanning but bypasses the processing step. Rather than a user notification being sent via email, a "Processing..." status gif would be displayed on the Resources view in the platform UI and a green checkmark (success) or red X (error, with failure message) would later appear next to the resource. Processing time would include the file upload duration plus the clamAV scan duration; this should not represent any noticeable delay over the current process. This workflow enables the user to continue uploading additional resources or perform other platform operations as desired without blocking.

A similar process would be implemented for project data exports:

1) User requests export of their project data via the platform UI;
    1a) Optionally, the user may choose from a list of other project members to share access to the export;
2) Platform code sends a message with the relevant data (project ID, requesting user, additional users, datestamp, etc) to an SQS queue;
    2a) On successful receipt, platform code logs the request details to a table in the platform database;
3) A scheduled process running on a worker node uses long polling to check the SQS queue and retrieve new messages;
4) The message is parsed for the request details;
5) The project export file is prepared;
    5a) If the export succeeds, it is saved to an S3 bucket and a notification is sent to the user(s) with a link to download the file;
    5b) If the export fails, platform admins are alerted and the requesting user receives a notification informing them of the outcome;
    5c) Status of the export request is written to the platform database;
6) The project data export will be available for a limited duration, with the proposed time being 72 hours;
    6a) Both the platform UI and the successful user notification will advise them of this limit;
    6b) The S3 bucket for storing exports will be configured to delete files after 72 hours;
    6c) If a user wishes to access a project export file after 72 hours, they must resubmit the export request.

Notes for both workflows:

All transport will occur over SSL. All S3 buckets will be encrypted.

S3 allows user-defined metadata, which we can configure with the project ID, resource ID, etc, as needed to facilitate tracking of files from the platform side.

Utilization of long polling enables the worker node to receive new queue messages as soon as they are available, eliminates the return of empty messages, and reduces the amount of polling required.

Worker node(s) for clamAV and file processing will be autoscaled dynamically to meet demand as necessary.

User notification will initially be an email, but methods may later expand to include SMS, native platform alert, Drift, etc. Admin notifications will initially be email, but may also include cross-posting to a Slack channel.

Logging all export requests should be considered non-optional, for security and auditing purposes in addition to providing feature performance metrics and debugging. Logging of import processes is less critical at this time, but could easily be implemented as well.

Optional: As an alternative to a worker node for processing, we could configure AWS Lambda to process files upon receiving the SQS message. This does give us less control and portability, however.

## Benefits

Through these new workflows, transport and processing workloads are decoupled from the platform server, which prevents performance impact to the platform during operation. By making the processes asynchronous, the user experience is improved by not forcing them to wait for the process to complete, and by allowing upload and download of larger file sizes with more generous timeout durations. Security scanning of files is introduced and ample room remains for adding new features and components to these processes in the future. The microservice architecture is dynamically scalable in the face of concurrent requests and intensive batch jobs, as well as being more resilient in recovering from errors. By configuring a term limit for all project export files, project data is protected from the potential security risks of indefinite storage, and storage costs to Cadasta are minimized for these large files.

## Consequences

There is the financial cost to Cadasta involved in implementing the worker node, though relatively small. There is also cost incurred in storing project export data, but is significantly reduced under this proposed workflow compared to implementing export as a platform-based feature.
