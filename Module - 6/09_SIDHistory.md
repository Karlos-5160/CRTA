## The basic idea

Think of a company with a **parent office** (headquarters) and a **child office** (branch). Active Directory works the same way — a "parent domain" and a "child domain" that trust each other automatically.

Every user account has an ID card called a **SID** (Security Identifier). Normally, your SID tells the network "which office you belong to and what you're allowed to touch."

There's a special hidden field on accounts called **SID History**. It was designed for a legitimate reason: when someone's account moves from one domain to another (say, an employee transfers offices), IT can add their *old* SID into this field so they don't lose access to their old resources during the transition. When they log in, the system checks both their current SID *and* everything in their SID History — treating the person as if they still belonged to that old domain too.

## Where the attack comes in

An attacker who has already fully compromised the **child domain** can abuse this "keep old access" feature to forge a ticket that says:

> "I'm Administrator in the child domain — *and* I also carry the SID history of the parent domain's Enterprise Admins group."

Since domains in the same forest trust each other by default and don't check whether that SID History is legitimate (this check is called **SID filtering**, and it's off by default within a forest), the **parent domain controller sees that Enterprise Admins SID and says "sure, come on in as an admin."**

That's the entire trick: smuggle a powerful group's SID into a field that's supposed to be for migration bookkeeping, and the trust relationship blindly honors it.

## Mapping it to your file's example

Your notes walk through exactly this, step by step:

1. **Get a foothold** in `child.warfare.corp` (via Pass-the-Hash) and confirm you have local admin there.
2. **Confirm the trust** — `warfare.corp` is the forest root/parent, `child.warfare.corp` is its child, and they share a two-way trust.
3. **Steal the child domain's `krbtgt` hash** (via DCSync) — this is the master key used to sign/forge Kerberos tickets for that domain.
4. **Grab both SIDs** — the child domain's SID and the parent domain's SID.
5. **Forge a Golden Ticket** for "Administrator" in the child domain, but tell Mimikatz to stamp the **parent domain's Enterprise Admins SID (ending in `-519`)** into the ticket's SID History (`/sids:<parent_domain_sid>`).
6. **Inject the ticket into memory** (`/ptt`) — now your session claims that hidden extra membership.
7. **Walk into the parent DC** — before injection, `dir \\dc01.warfare.corp\C$` gets denied; after injection, it works, because the parent DC's PAC check sees the Enterprise Admins SID sitting in your ticket's history and grants forest-wide admin rights.

So in one sentence: **you never actually joined the "Enterprise Admins" group — you just handed the parent domain a ticket claiming your account used to carry that membership, and because intra-forest trust doesn't verify that claim, it's accepted at face value.**

**Defensive note** (since your notes end at the attacker's win): the fix is enabling **SID filtering / quarantining** on the trust, which strips out SID History claims that don't match the domain that issued the ticket — closing exactly this loophole.