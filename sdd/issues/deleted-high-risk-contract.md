# A deleted high-risk contract asks three questions

`check()` in `template/sdd/bin/sdd` walks the contracts of the staged tree only. A batch that deletes or renames a
contract declaring `risk: high` never reads it, so its quiz asks three questions instead of five, and `audit` does not
recommend a second reading. Seen by an automated review of the commit "Ask for a quiz by the risk of the batch".

Not a lock to close (everything can be bypassed), but such a batch is exactly one to read twice. Fix: also walk the
base's contracts that the tree lacks, and judge them with `risky "$base:$contract"`.
