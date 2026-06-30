# Code Review & the Maintainer Mindset

## Task1 

**Find three open PRs. Leave substantive comments on each, using Conventional Comments labels. Substantive means design or correctness, not formatting.**

- First [PR](https://github.com/celery/celery/pull/10359): I think this PR is was little bit of a rush given.

- Second [PR](https://github.com/httpie/cli/pull/1883):
 The requests library (which httpie uses under the hood) automatically un-squishes gzip/deflate files for you as they download. So by the time httpie counts the bytes it received, it's counting the un-squished, much bigger version, maybe 10,000 bytes The old, buggy code compared:

  That produced the real-world bug people reported: `http --download` claiming a perfectly good gzip download was "incomplete," when it wasn't.

  This PR exists to stop httpie from wrongly comparing compressed-size to decompressed-size and crying wolf about a fine download.
My comment, in that context, was flagging: "your 'skip the check' rule fires for any compression type — but for one specific type (br), the un-squishing might not actually happen if a tool is missing, so in that one case, skipping the check throws away a comparison that would've actually been valid and useful."    
That's it — it's not "this is definitely broken," it's "here's a specific case where I think it might quietly stop catching a real problem, can you verify?"



## Task 2 

**Self-review write-up. Take one of your own past PRs (the one you’re least proud of).
Pretend you’re the reviewer. Write what you would have said. Write about it in your weekly
journal.**


* As my peers did, I decided as well to review the same [PR](https://github.com/wasmerio/Python-Scripts/pull/564#pullrequestreview-4598634725) I did for JOSA. 
* Mistakes I mentioned: 

   1) A real bug I didn't notice was what if we still had website lists from a previous sessions, `self.website` just remove websites from the current session.
   2) `working_hours` logic wasn't give option for users to set for weeks or months, or no specific timezones.
   3) block_websites checks if entry not in content, which guards against duplicate lines within a single file, but it appends without ever sorting or namespacing the block, so the hosts file accumulates whatever ordering happens to occur from prior runs with different site lists. 
