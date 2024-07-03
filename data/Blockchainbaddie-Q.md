L-01


Function: `openCdpWithEth`
Audit Findings:

Parameter Validation: Validate all input parameters to ensure they are within acceptable ranges. 

Emit Event: Ensure the event ZapOperationEthVariant is correctly emitted and contains all necessary information.


C-01
Reentrancy Protection: Ensure `_openCdp` is protected against reentrancy attacks by using the nonReentrant modifier. 



L-02

Function: `openCdpWithWstEth`
Audit Findings:

Parameter Validation: Validate all input parameters to ensure they are within acceptable ranges.

Emit Event: Ensure the event `ZapOperationEthVariant` is correctly emitted and contains all necessary information. 


C-02
Reentrancy Protection: Ensure `_openCdp `is protected against reentrancy attacks by using the nonReentrant modifier.




L-03
Function: `receive`
Audit Findings:

Address Validation: Ensure the wrappedEth address is correctly set and cannot be manipulated. 

Function: `openCdpWithWrappedEth`
Audit Findings:


Parameter Validation: Validate all input parameters to ensure they are within acceptable ranges. 
Emit Event: Ensure the event ZapOperationEthVariant is correctly emitted and contains all necessary information. 


Function: `openCdp`
Audit Findings:

Parameter Validation: Validate all input parameters to ensure they are within acceptable ranges.
Emit Event: Ensure the event `ZapOperationEthVariant` is correctly emitted and contains all necessary information.








