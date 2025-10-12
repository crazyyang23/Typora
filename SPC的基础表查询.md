select top 1000 * from Spc_SpcStat(nolock) where SpcConfigID='cc536fd3-919e-4f60-919b-0849ffdc6d07' ORDER BY MonitorTime DESC

 

select top 1000 * from spc_SampleAndStat(nolock) where spcstatId='2813f0eb-35c9-4299-93fa-f9e3355c9de5'

 

SELECT TOP 100 * FROM spc_Sample WITH(NOLOCK) WHERE SampleGroupID='dc8527e3-29bc-42de-9dce-b41c67007ea4' 