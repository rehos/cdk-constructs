![cloudcomponents Logo](https://raw.githubusercontent.com/cloudcomponents/cdk-constructs/master/logo.png)

# @cloudcomponents/cdk-codecommit-backup

[![Build Status](https://travis-ci.org/cloudcomponents/cdk-constructs.svg?branch=master)](https://travis-ci.org/cloudcomponents/cdk-constructs)
[![typescript](https://img.shields.io/badge/jsii-typescript-blueviolet.svg)](https://www.npmjs.com/package/@cloudcomponents/cdk-codecommit-backup)

> Backup CodeCommit repositories to S3

## Install

```bash
npm i @cloudcomponents/cdk-codecommit-backup
```

## How to use

```typescript
import { Construct, Stack, StackProps, Duration } from '@aws-cdk/core';
import { Repository } from '@aws-cdk/aws-codecommit';
import { Schedule } from '@aws-cdk/aws-events';
import { SnsTopic } from '@aws-cdk/aws-events-targets';
import { Topic } from '@aws-cdk/aws-sns';
import { EmailSubscription } from '@aws-cdk/aws-sns-subscriptions';
import {
  BackupBucket,
  S3CodeCommitBackup,
} from '@cloudcomponents/cdk-codecommit-backup';

export class CodeCommitBackupStack extends Stack {
  constructor(scope: Construct, id: string, props?: StackProps) {
    super(scope, id, props);

    const repository = Repository.fromRepositoryName(
      this,
      'Repository',
      process.env.REPOSITORY_NAME as string,
    );

    const backupBucket = new BackupBucket(this, 'BackupBuckt', {
      retentionPeriod: Duration.days(90),
    });

    // The following example runs a task every day at 4am
    const backup = new S3CodeCommitBackup(this, 'S3CodeCommitBackup', {
      backupBucket,
      repository,
      schedule: Schedule.cron({
        minute: '0',
        hour: '4',
      }),
    });

    const backupTopic = new Topic(this, 'BackupTopic');

    backupTopic.addSubscription(
      new EmailSubscription(process.env.DEVSECOPS_TEAM_EMAIL as string),
    );

    backup.onBackupStarted('started', {
      target: new SnsTopic(backupTopic),
    });

    backup.onBackupSucceeded('succeeded', {
      target: new SnsTopic(backupTopic),
    });

    backup.onBackupFailed('failed', {
      target: new SnsTopic(backupTopic),
    });
  }
}
```

## Example

See more complete [examples](https://github.com/cloudcomponents/cdk-constructs/tree/master/examples).

## License

[MIT](./LICENSE)
