# PostingModel base case

this covers the posting model of EPICS

when performing cross border we have to post to the following item

# direction US -> Canada
1. FTM perform Sanction check
2. FTM send request to EPICS with the following legs
    - Debit Customer Account on Finacle US
    - Credit ICA Account GUS.IBD on Finacle US
3. EPICS response completed via API
4. FTM send request to EPICS with the following legs
    - Debit ICA(HOU) on Finacle IBD
    - Credit Suspense account on Finacle IBD
    - Debit DDA pool account on DDA
    - Credit Customer Account on DDA
    - Debit SG FNCL Offset on SG
    - Credit DDA Pool  on DDA
    - Credit Fin offset on SG
    - Debit ICA(HOU) on SG
5. EPICS Orchestrate and perform first half of the request and send restful posting request to FPS the one that are on DDA and SG
6. if FPS response ok from API will proceed with posting to Finacle for the Finacle IBD request. if API also response ok,it will trigger a response to FTM saying 200 success

5 and 6 are sequential if 5 and 6 both success EPICS will response success to FTM
if 5. response timeout will stop process and wait for Kafka. if Kafka does not come in 2 minutes will treat as fail. Failure will trigger a error kafka event send to FTM

For any failure from API ,either timeout response, FTM will receive response 202 accepted. and success or failure will be based on Kafka event

if 5 response success via kafka, will continue the 6 logic, but will fire kafka success or fail to FTM,
if 6 fail will trigger reversal post to FPS on those legs. and will wait for success or fail from API or Kafka. if the reversal success nothing happen. if the reversal fail, should triggers an alert?

