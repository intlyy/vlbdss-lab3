So far, we have built a set of monitoring and alert service.
Now we check whether the service has sufficient availability, and then think about the solution.

1. Suppose there is a bug in vm-operator, and the memory is exhausted infinitely, which will cause no memory available for other machines deployed on the same EC2 machine. How to avoid this situation?

2. Suppose there is a bug in vmstorage, there is a small probability that it will crash, so how do we ensure the overall service availability? What if all VMInserts are on the same EC2 when one EC2 fails?

3. If the availability of the indicator collection vmagent and the alarm service vmalert is low, how can we avoid the problem of customer A from affecting customer B.

4. If all VMCluster services crash for 5 minutes, customers cannot query metric data during this time. But what about after service is restored? Is there any chance for this part of the customer's data to be recovered?

Please answer questions 1 to 4 and fill in answer/1 to answer/4, then submit code to GitHub.
Describe your solution in answer.txt, if it is correct, you can get 10 points, if you modify the code correctly, you can get another 5 points. See answer/example
