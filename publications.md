---
layout: page
title: Publications
---

### [A Graph-based Dataset of Commit History of Real-World Android apps](/public/publications/MSR_2018.pdf)

![Schema of the graph database containing commit history of Android apps](/public/img/dbstructure.png)

Obtaining a good dataset to conduct empirical studies on the
engineering of Android apps is an open challenge. To start
tackling this challenge, we present AndroidTimeMachine, the first,
self-contained, publicly available dataset weaving spread-out data
sources about real-world, open-source Android apps. Encoded as a
graph-based database, AndroidTimeMachine concerns 8,431 real
open-source Android apps and contains: (i) metadata about the
apps’ GitHub projects, (ii) Git repositories with full commit history
and (iii) metadata extracted from the Google Play store, such as
app ratings and permissions.

* <cite>
    Franz-Xaver Geiger, Ivano Malavolta, Luca Pascarella, Fabio Palomba, Dario Di Nucci, Ivano Malavolta, and Alberto Bacchelli.
    2018.
    A Graph-based Dataset of Commit History of Real-World Android apps.
    In <em>Proceedings of the 15th International Conference on Mining Software Repositories, MSR.</em>
    ACM, New York, NY.
  </cite>
  [Bibtex](/public/publications/MSR_2018.bib)
* Presented at the [data showcase track of MSR 2018](https://2018.msrconf.org/event/msr-2018-data-showcase-papers-a-graph-based-dataset-of-commit-history-of-real-world-android-apps).
* Data is available in [Docker containers](/dockerImages) containing the Neo4j graph database and a Gitlab instance.
* [Replication package](https://github.com/AndroidTimeMachine/open_source_android_apps)
* [PDF download](/public/publications/MSR_2018.pdf)


### [Self-Reported Activities of Android Developers](/public/publications/mobileSoft_2018.pdf)

![Self-reported activities of Android Developers](/public/img/self-reported-activities.png)

To gain a deeper empirical understanding of how developers work
on Android apps, we investigate self-reported activities of Android
developers and to what extent these activities can be classified
with machine learning techniques. To this aim, we firstly create
a taxonomy of self-reported activities coming from the manual
analysis of 5,000 commit messages from 8,280 Android apps. Then,
we study the frequency of each category of self-reported activities
identified in the taxonomy, and investigate the feasibility of an
automated classification approach. Our findings can inform be used
by both practitioners and researchers to take informed decisions or
support other software engineering activities.

* <cite>
    Luca Pascarella, Franz-Xaver Geiger, Fabio Palomba, Dario Di Nucci, Ivano Malavolta, and Alberto Bacchelli.
    2018.
    Self-Reported Activities of Android Developers.
    In <em>5th IEEE/ACM International Conference on Mobile Software Engineering and Systems.</em>
    ACM, New York, NY.
  </cite>
  [Bibtex](/public/publications/mobileSoft_2018.bib)
* Presented at [MobileSoft 2018](https://www.icse2018.org/event/mobilesoft-2018-papers-self-reported-activities-of-android-developers).
* [Replication package](https://figshare.com/articles/Self-Reported_Activities_of_Android_Developers/5802909)
* [PDF download](/public/publications/mobileSoft_2018.pdf)
