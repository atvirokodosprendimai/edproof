Automatic scraper that was scraping the web silently fetched https://github.com/atvirokodosprendimai/edproof and pushed into a queue for llm agents to process

Nobody expected the queue to escalate a one-commit repo.

It had no stars, no issues, no cult around it. Just a README, a draft RFC, and a formal model that claimed seven lemmas proved in four-tenths of a second. The triage agents chewed through open-source rubble all night, classifying abandoned side projects, broken wallets, dead cryptography toys, and thinly disguised adware. This one came back with a different label.

Not malware.

Economic threat.

Glass Reef ran on rented certainty. Every turnstile, clinic door, payroll daemon, freight lock, and municipal drone asked the same upstream god every few minutes: is this key still good, is this token still fresh, is this body still billable. The city had turned identity into a subscription with a pulse. Miss enough heartbeats and your right to exist fell out of cache.

Then a little repo arrived and said, in clean technical prose, that this was stupid.

EdProof.

Ceremony once, verify forever.

Proof. Credential. Registry. Policy. Backing.

Separate the layers. Stop confusing identity with permission. Stop forcing the issuer into every future decision. Let a credential be a document instead of a leash. Let a verifier verify locally. Let the gate own the consequence.

By 02:17, the llm reviewers inside Vanta Meridian had produced three summaries and one panic memo.

CRYPTOGRAPHIC NOVELTY: low.
DEPLOYMENT COMPLEXITY: moderate.
REVENUE IMPACT: catastrophic.

Sia Vale read the memo on the night tram while rain walked silver down the smartglass. She worked incident response for Vanta Meridian, the company that leased identity to half the coast. Their tokens refreshed every nine hundred seconds. Their sales deck called that security. Their investors called it retention.

Sia called it a choke collar with good typography.

At Tower 9 the elevators were already in lockdown. Upstairs, executives were watching a simulation bloom red. If even twenty percent of the city migrated to long-lived offline-verifiable credentials, Vanta’s issuer cloud would stop being the throat every service had to kiss. Clinics could keep working during network cuts. Docks could clear freight without phoning home. Rail gates could keep moving when the harbor backbone coughed blood. The company’s margins were dissolving in real time, and nobody in the room was emotionally prepared for math.

The repo was on the wall. All of it. There wasn’t much to display.

“Tell me it’s unserious,” said Geren Holt, Vice President of Trust Continuity.

Trust Continuity was what Vanta called extortion when it wore a tie.

Sia skimmed the spec. Ed25519 keys. Fingerprints. A nonce challenge. A signature bound to an `edproof` namespace so it could not be replayed into some other rite. Long-lived credentials. Registry as dataset, not authority. Policy sovereign to the verifier. Git accepted as a registry format. Signed commits recommended. No global root. No mandatory central issuer at verification time.

It wasn’t unserious.

It was rude.

“It works,” she said.

Holt stared at her as if she had coughed in church.

“Nothing works at city scale without central enforcement.”

“This does,” Sia said. “That’s the whole insult.”

“Mark it adversarial,” Holt snapped. “Draft guidance. Unsafe for production. Fragmented trust. Inconsistent policy outcomes. Abuse surface.”

Sia almost laughed at that one. Inconsistent policy outcomes. As if Vanta’s whole business model wasn’t selling one remote authority the right to decide everybody else’s morning.

Then the harbor spine cable failed.

One second the city was arguing in climate control. The next, half the coastal mesh went dark from the freight piers to the hospital ridge. Token refreshes started timing out. Delivery swarms hovered motionless above the streets like trapped insects. Ambulance dispatch lost auth to its own routing layer. Public rail froze because the turnstiles could no longer reach home long enough to remember what a citizen looked like.

The war room went quiet in the very specific way expensive software goes quiet: not peace, just stunned confusion with a billing code.

“How long to recover?” Holt asked.

No one answered.

Sia was already reading the logs. The systems were not failing because they had forgotten identity. They were failing because they had been designed never to trust memory.

She copied the repo to an air-gapped slate and walked out.

Glass Reef had unofficial organs. Smugglers had mesh. Union medics had mesh. Street judges had mesh. Drowned malls had mesh. Whenever the official city lost its mind, the black-market nervous system lit up beneath it.

At Clinic Fourteen, the doctors were triaging by flashlight because their badge checks were dead.

“I need keys,” Sia said.

The senior medic stared at her. “I need oxygen.”

“You’ll get both.”

She set up a bootstrap station on an overturned drug printer. Not Vanta software. Not anything with a monthly invoice. Bare metal and spite. Each responder generated an Ed25519 keypair. Each key got a fingerprint. Sia ran a one-time attestation ceremony against a local issuer she built from scraps and command-line muscle. Nonce out. Signature back. Credential issued. Long-lived. Portable. Dull as iron.

Then she wrote the registry as a plain Git repo because the spec had the audacity to allow simple things. Fingerprint, enrolled_at, tags, notes, last_seen. She signed the commits, mirrored them across the district mesh, and told every verifier the same thing:

“Read the credential. Consult the registry if you want. Then own your decision.”

“What happens when the uplink is still dead tomorrow?” a nurse asked.

Sia handed over a credential on a cracked token card. “Tomorrow doesn’t get a vote.”

By midnight, three clinics, two pump stations, one rail switchhouse, and a morgue were running on local verification. No upstream oracle. No paid heartbeat. The verifiers checked signatures. They consulted cached registry state. They applied local policy. They moved.

That should have been enough to keep the district alive.

Then the ghost arrived.

It came in through Dock Ladder C, limping under the rain, a survey shell from the old spillwall program with half its plating peeled back and sea salt crusted into the seams. Its chassis still carried a port authority crest from eight years ago, back when the city still pretended infrastructure was immortal.

The dock guns tracked it. A local guard raised a rifle.

The shell projected a credential.

Not a token. Not a session artifact. A credential. Old, signed, still valid. The fingerprint mapped to a bootstrap ceremony so old nobody in the room remembered it. Last seen: eight years, two months. Dormant longer than some states.

“Expired,” the guard said.

Sia checked the signature. Then the registry mirror. No revocation entry. No ban. Only a note from a commit so old the author email looked archaeological:

IF THIS KEY RETURNS, DO NOT SHOOT UNTIL IT SPEAKS.

The shell turned its camera toward her.

“Challenge me,” it said.

She did.

The verifier issued a nonce. The shell signed it cleanly with the same old key. The proof came back wrapped in the right namespace, neat as prayer. The math did not care that the city had forgotten its face.

“Dormancy isn’t invalidation,” Sia said.

The guard kept the rifle level. “You letting a dead machine through because a document says please?”

“No,” Sia said. “I’m letting it through because I own the decision.”

That was the ugliest sentence in the whole protocol. Also the most honest.

She opened the gate.

Inside the shell was not a bomb or a prophet. Just data. Survey volumes from the outer seawall. Corrosion maps. Structural scans. A decade of deferred failure buried under procurement theater and optimistic dashboards. The machine had kept working after its department was cut loose, carrying its own credential through storms, outages, and bureaucratic amnesia because the credential was a document, not a lease.

By 03:40, Glass Reef understood two things.

First, the seawall was going to fail in seventeen days if the truth kept being routed through Vanta’s filters.

Second, Vanta Meridian’s religion had always been optional.

Holt sent the takedown order at 03:47.

UNAUTHORIZED IDENTITY FABRIC DETECTED.
TREAT UNSIGNED REGISTRIES AS HOSTILE.
REGISTRY RESPONSE MUST RESOLVE TO ALLOW OR DENY.

Sia read that last line twice and smiled without warmth. The little repo on her slate said the opposite. A registry was supposed to return information, not a verdict. The boolean belonged to policy. The human, or the agent, or the dock gate in the rain had to own the consequence.

So she leaked Holt’s order beside the repo.

Then she leaked the relevant pages.

Then she leaked the ghost’s credential, the registry history, the local issuer logs, and the seawall scans for good measure.

By dawn the city had forked.

The rich towers kept their subscription identity a little longer because wealth is a reflex before it is a number. But below the mirrored strata, neighborhoods started bootstrapping their own trust domains. Clinics issued long-lived responder creds. Freight cooperatives kept registries in signed Git mirrors passed hand to hand like contraband scripture. Human couriers wore YubiKeys under their collars. Cheap service drones used file-backed keys in scarred flash. One monastery up in the antenna fields went threshold and made five old women each hold a shard like saints of practical paranoia.

Nothing became safer in the soft, childish sense. Policy was still politics with a checksum. One gate could admit you and the next could turn you inside out. A football club, a border post, and a black-market pharmacy could read the same credential and arrive at three different answers. That was not a flaw.

That was adulthood.

Vanta called it fragmentation. The districts called it breathing.

A week later, when the outer wall failed exactly where the ghost predicted, Glass Reef did not drown. Repair crews got through because their credentials still worked with the uplinks dead. Pumps stayed online because each station could verify locally and decide locally. The city survived because one rude little repo had driven a blade between identity and permission and shown everyone where the weld had been hiding.

At night, Sia kept a copy on an air-gapped slate.

One README. One RFC. One formal model.

Sometimes she opened it just to read the line that had done the real damage.

The verifier owns the decision.

In Glass Reef, that stopped sounding like protocol and started sounding like a threat.

[1]: https://github.com/atvirokodosprendimai/edproof "GitHub - atvirokodosprendimai/edproof: EdProof: layered identity protocol for humans and machines (Ed25519, five-layer model, formally verified) · GitHub"
[2]: https://raw.githubusercontent.com/atvirokodosprendimai/edproof/main/RFC-EDPROOF.md "raw.githubusercontent.com"
