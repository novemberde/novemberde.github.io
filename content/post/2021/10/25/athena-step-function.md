---
title: "Athena & Step Functions 로 통계 파이프라인 구축하기"
tags: [aws, athena, stepfunctions]
date: "2021-10-25T11:30:03+00:00"
ShowBreadCrumbs: true
ShowReadingTime: true
ShowPostNavLinks: true
---

보통 Airflow 와 EMR 조합으로 통계 파이프라인을 관리하곤 한다.
사내에서 빠르게 통계를 구축하고 관리하기 위해 불필요한 인프라 관리를 제외하고
pipeline에 대해서만 집중할 수 있도록 step functions를 도입하고, 이 경험에 대해서 공유해보았다.

---

## Youtube


<iframe width="560" height="315" src="https://www.youtube.com/embed/MS7CulWSc2g" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Slides

<iframe src="//www.slideshare.net/slideshow/embed_code/key/55KQbsx2fbDEqI" width="595" height="485" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC; border-width:1px; margin-bottom:5px; max-width: 100%;" allowfullscreen> </iframe> <div style="margin-bottom:5px"> <strong> <a href="//www.slideshare.net/awskr/athena-step-function-aws-community-day-online-2021" title="Athena &amp; Step Function 으로 통계 파이프라인 구축하기 - 변규현 (당근마켓) :: AWS Community Day Online 2021" target="_blank">Athena &amp; Step Function 으로 통계 파이프라인 구축하기 - 변규현 (당근마켓) :: AWS Community Day Online 2021</a> </strong> from <strong><a href="//www.slideshare.net/awskr" target="_blank">AWSKRUG - AWS한국사용자모임</a></strong> </div>

## References

- [https://youtu.be/MS7CulWSc2g](https://youtu.be/MS7CulWSc2g)
- [https://www.slideshare.net/awskr/athena-step-function-aws-community-day-online-2021](https://www.slideshare.net/awskr/athena-step-function-aws-community-day-online-2021)
