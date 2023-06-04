---
title: Doctrine select to iterable with batch
slug: doctrine-select-to-iterable-with-batch
date: 2023-06-04 00:00:00+0000
categories:
    - dev
tags:
    - php
    - symfony
    - doctrine
---

I was looking for examples on how to do processing on possibly a lot of result (millions) from a doctrine select operation.
Then I stumbled upon [a stackoverflow question](https://stackoverflow.com/questions/23545768/doctrine-batch-processing-iterate-high-memory-usage), [a blog page](https://hubofco.de/php/2019/04/19/how-to-update-millions-of-records-with-doctrine-orm/), and [doctrine docs](https://www.doctrine-project.org/projects/doctrine-orm/en/2.14/reference/batch-processing.html#iterating-large-results-for-data-processing). While it helped a lot, the stackoverflow answer and the blog page is using functions from older doctrine version. And I decide to post my own 'updated' version.

I am using pinned version of each library for my projects. So here is my current composer.json version at the time of this writing: `(symfony = v5.4.*, doctrine/orm = 2.14.1, doctrine/doctrine-bundle = 2.9.0)`.

```php
printf("Starting with memory usage: %d MB\n", \memory_get_usage(true) / 1024 / 1024);

$batchSize = 1000; // flush for every batch-size
$numberOfRecordsPerPage = 5000; // number of records for each SQL query
$totalRecordsProcessed = 0;
$totalRecords = (int) $this->entityManager
    ->createQuery('SELECT COUNT(u.id) FROM App\Entity\User u')
    ->getSingleScalarResult()
;

while (true) {
    $myQuery = $this->entityManager
        ->createQuery('SELECT u FROM App\Entity\User u')
        ->setMaxResults($numberOfRecordsPerPage)
        ->setFirstResult($totalRecordsProcessed)
    ;

    /** @var iterable $iterableResult */
    $iterableResult = $myQuery->toIterable();
    if (empty($iterableResult->current())) {
        break;
    }

    foreach ($iterableResult as $row) {
        $user = $row;

        // do stuff with the data in the row

        // detach from Doctrine, so that it can be Garbage-Collected immediately (*)
        $this->entityManager->detach($row[0]);

        if (($totalRecordsProcessed % $batchSize) === 0) {
            $this->entityManager->flush();
            $this->entityManager->clear();
        }

        $totalRecordsProcessed++;
    }

    if ($totalRecordsProcessed === $totalRecords) {
        break;
    }
}

$this->entityManager->flush();
$this->entityManager->clear();

printf("Ending with memory usage: %d MB\n", \memory_get_usage(true) / 1024 / 1024);

```
