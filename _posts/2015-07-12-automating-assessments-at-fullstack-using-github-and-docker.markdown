---
layout: post
title: "Automating Assessments at Fullstack Using GitHub and Docker"
date: 2015-07-12
---

As a Teaching Fellow at Fullstack Academy, I am required to complete a thesis
project of my choosing. Another fellow, [Christian
Cueto](https://www.linkedin.com/in/christianmcueto), and I decided to team up
on our theses and streamline/automate the assessment process at Fullstack.

When we were students taking assessments at Fullstack, the only way to submit
assessments was by forking the assessment repository on GitHub, doing our work
on our local machines, pushing to our GitHub repos, going to GitHub.com to
download the ZIP file of our repos, then emailing that ZIP file to the
instructors. Hardly a technically progressive process, especially for a
progressive coding bootcamp.

But the grading process for the instructors was even worse. They would receive
an email from each of the students (25 students total in our cohort), each
containing a ZIP file. They would have to download the ZIP, decompress it, run
the install procedures (npm install, bower install, etc.), run the test
command, then manually record the scores for each student in a Google
spreadsheet. That sounds terrible.

This is why we decided to automate this process with a project we call
AssessDot.

We have not yet completed AssessDot, but I'm going to walk you through the
architecture of the project.

## Assessment Creation

An instructor creates an `assessment` using a GitHub repository. Assessments
actually consist of multiple "steps", but for these diagrams we have simplified
things, and assessments consist of only a GitHub repo.

When the instructor administers the assessment to a cohort, a
`cohortAssessment` document is created. This cohortAssessment consists of a
reference to the assessment, a reference to the cohort, and the start and end
times that the students can take the assessment. Administering an assessment
also creates a `studentSolution` document for each student in the cohort. A
studentSolution represents the student's assessment score and responses to the
assessment; in this simplified case, the URL of the student's GitHub repo.

Creation of a cohortAssessment grants the students in the cohort access to view
the assessment on Learndot, Fullstack's internal educational platform. The
reason for having both assessments and cohortAssessments is because assessments
are administered to each new cohort. So instead of recreating the same
assessment for each cohort, the cohort is given access to the assessment via
the cohortAssessment.

## GitHub Integration

For students, taking an assessment under this new system is much the same as
before. Students will still fork the assessment repository, clone to their
machines, and write their code in the local dev environments. But when they
submit, all they will need to do is simply `git push` to their GitHub repos.
AssessDot will handle everything after that.

This is done using GitHub webhooks. When a student forks the main assessment
repo (which will be done via the Learndot application), AssessDot will create a
webhook on the student's repo. This webhook will ping the Learndot server and
inform us anytime that student pushes to that GitHub repo.

## Docker Containers

When the Learndot server is informed of a push to a student's assessment repo,
we extract relevant info from the webhook payload, such as the student's GitHub
username and the repo URL that was pushed to. The Learndot server sends this
information to a separate Docker server which will spin up a Docker container
from an image that we've already setup. The Docker container clones the
student's repo, runs the tests, and calculates the student's score. The
student's results are then sent to Learndot and the Docker container is shut
down.

The need for Docker is to create an environment to be able to safely run the
students' code. Without this contained environment, our server would be
extremely vulnerable to attacks both intentional and unintentional.
