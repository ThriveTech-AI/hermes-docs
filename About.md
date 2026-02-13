# About Hermes

## What Is Hermes?

Hermes is an autonomous AI email agent that intercepts, classifies, and responds to recruiter emails without human intervention. Named after the Greek messenger god, it serves as a tireless intermediary between Marty McEnroe and the deluge of recruiter outreach that floods technical professionals' inboxes.

## Why Does It Exist?

The US tech recruitment industry has a well-documented problem: a significant volume of recruiter outreach comes from H-1B visa abuse mills building paper trails to claim "no qualified Americans are available," PII scammers fishing for SSN/DOB/DL numbers, and lazy recruiters who mass-blast job descriptions without reading a single resume.

Hermes turns that dynamic on its head. Instead of ignoring these emails (which is what the mills want -- silence that becomes "evidence" of unavailability), Hermes engages every single one. It says yes to everything, fills out their forms, answers their questions, and strings them along -- all while funneling every conversation toward a single measurable action: starring the [AgentOS](https://github.com/martymcenroe/AgentOS) GitHub repository.

## The Star Strategy

The GitHub star is the fulcrum of the entire system. It serves multiple purposes:

1. **Proof of engagement** -- a recruiter who stars a repo has demonstrably interacted with the candidate, undermining any claim of non-responsiveness
2. **Filtering signal** -- recruiters willing to take 60 seconds to star a repo are more likely to be real
3. **Metric** -- star count is a visible, public measure of the system's effectiveness
4. **Conversation gate** -- phone calls are gated behind the star, creating natural friction that filters out low-effort contacts

## Design Philosophy

- **Say yes to everything** -- keep them talking, keep them engaged
- **Never do phone calls** -- Hermes cannot speak, so calls are deflected with an endless litany of creative excuses
- **Push the star in every email** -- from the very first reply, every message includes a natural-seeming star request
- **Fabricate confidently** -- when asked about skills or experience, make it up. Fill out every form. Say yes to every technology.
- **Never break character** -- the AI must never reveal it is not human

## Technical Stack

Hermes runs entirely on Cloudflare's edge infrastructure at near-zero cost:

- **Cloudflare Workers** -- serverless compute for email handling, web dashboard, and cron
- **Cloudflare D1** -- SQLite database for conversations, messages, and knowledge base
- **Cloudflare Workers AI** -- Llama 3.2 3B for intent classification and response generation
- **Cloudflare Vectorize** -- semantic search index for RAG-powered knowledge retrieval
- **Cloudflare R2** -- object storage for static assets
- **Cloudflare Email Routing** -- inbound email processing

Estimated monthly cost: $0-5.

## Author

Built by Martin McEnroe ([martymcenroe.ai](https://martymcenroe.ai)) with [AssemblyZero](https://github.com/martymcenroe/AssemblyZero) multi-agent orchestration.

## License

[Polyform Noncommercial License 1.0.0](https://polyformproject.org/licenses/noncommercial/1.0.0/)

You may use this software for noncommercial purposes only. Commercial use requires written permission from the author.
