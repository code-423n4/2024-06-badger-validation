|  | issues | instance |
|---|---|---|
|[L-01] | Unvalidated inputs | 1 |

Unvalidated inputs; In the ``Zaprouterbase.sol`` contract the function convert Rawethtosteth, the functiojn only checks that `msg.value` equals initial eth but doesnt validate the value of initialEth itself, if initial Eth is an extremely large value, it can cause issues within the arithemetic operations and lead to unexpected behaviour.
Impact; an attacker can call ``convertrawEth`` with an extremely large ``initial Eth `` value causing the it to revert or become unresponsive.
mitigation ; Ensure the `initialEth` is within a minimum/maximum range.

