# Reading list: tuning a model to be a house-building expert for newbies

Compiled 2026-11-11. All URLs verified with curl (HTTP 200) on this date, except
the six in the last section, which block automated checks.

Aimed at someone who has already done the text-SFT pass of example 1. The root
references.md covers the generic toolchain (TRL/PEFT, LoRA, MLflow, the eval
guidebook, Argilla/distilabel, serving), and the "Do you even need to
fine-tune?" page is this example's required first stop. The closest sibling is
projects/4-restoration-ecology/reading.md - same RAG-versus-fine-tuning
decision and the same RAG mechanics, different corpus; that list goes deeper on
the build itself, and this one leans on it rather than repeating everything.

plan.md's framing is the spine here too: RAG over curated sources is the
primary pattern, fine-tuning is the complement for the explainer's voice and
format, and the genuinely new work is the safety-critical edge - wrong advice
can fail an inspection, waste five figures, or hurt someone. So the list is
weighted toward sources (what gets retrieved) and evaluation (how you know an
answer is safe), not training recipes.

Scope assumption: a US single-family new build under the International
Residential Code, with the author as the newbie asking about permits,
sequencing, trades, inspections, costs, and when to stop and hire a
professional. The California sources (owner-builder rules, Title 24, Monterey
County) match the restoration project's location; swap them for the actual
jurisdiction once the build site is known.

## 1. The framing - facts by retrieval, voice by the adapter

- https://arxiv.org/abs/2312.05934 - Ovadia et al., Fine-Tuning or Retrieval? - the head-to-head this example's default assumption rests on: retrieval consistently beats fine-tuning for injecting factual recall into small models, and construction facts (code sections, spans, sequences, costs) are exactly that
- https://arxiv.org/abs/2305.18997 - GPT Models in Construction Industry - expert review plus case studies of what general LLMs already do for construction tasks; the baseline-measurement study for what a general model knows before anything is tuned
- https://pmc.ncbi.nlm.nih.gov/articles/PMC10148230/ - JAMA Internal Medicine: licensed professionals rated ChatGPT's answers to real patient questions against physicians' - the closest published analog to "advice where being wrong has a cost", and a template for blind, expert-graded evaluation

## 2. RAG - the primary pattern, citation-first

- https://arxiv.org/abs/2005.11401 - Lewis et al., the original RAG paper; the formulation every later "retrieval-grounded advisor" inherits
- https://huggingface.co/learn/cookbook/en/rag_zephyr_langchain - HF Cookbook: a small open model plus a local retriever, runnable end-to-end; the closest recipe to what this example ships
- https://www.pinecone.io/learn/chunking-strategies/ - chunking strategies; the code is hierarchical numbered text (R507.2 lives under R507), so chunks must follow section boundaries and never split mid-clause
- https://www.anthropic.com/news/contextual-retrieval - contextual retrieval: prepend chunk context before embedding; a code subsection quoted alone is meaningless without its parent heading
- https://www.pinecone.io/learn/hybrid-search-intro/ - hybrid BM25 + dense retrieval; "what does R507.2 require" is an exact-phrase lookup, a keyword problem the dense retriever alone will fumble
- https://docs.llamaindex.ai/en/stable/examples/query_engine/citation_query_engine/ - LlamaIndex's citation query engine: answers that quote the code section back, the shape every answer from this model should take
- https://www.llamaindex.ai/blog/mastering-pdfs-extracting-sections-headings-paragraphs-and-tables-with-cutting-edge-parser-faea18870125 - parsing PDFs with structure preserved: span tables and assembly drawings are where the code's real answers live, and naive text extraction destroys them

## 3. The field - building science and construction practice

- https://buildingscience.com/document-search - Building Science Corporation's searchable library of digests and articles; moisture, air sealing, walls, roofs - the physics the model must not get wrong, free from the field's most-cited source
- https://www.prettygoodhouse.org/ - the Pretty Good House framework and book: pragmatic, science-based spec targets for a normal-budget house; the philosophy this model should explain instead of green-certification marketing
- https://www.greenbuildingadvisor.com/ - Green Building Advisor: a detail library plus an expert Q&A archive that reads like the target user's question log
- https://www.finehomebuilding.com/ - Fine Homebuilding: practitioner how-tos across every trade; the "how it's actually done" complement to the code's "what is required"
- https://www.jlconline.com/ - Journal of Light Construction: trade-level detail, including the sequencing and jobsite realities most newbie questions are really about
- https://www.energyvanguard.com/blog/ - Energy Vanguard: accessible building-science writing on insulation, ducts, and HVAC sizing; the best bridge from newbie intuition to physics

## 4. Jurisdiction - the IRC, amendments, permits, and owner-builder rules

- https://up.codes/ - UpCodes: free, navigable IRC plus state and local amendments; the everyday way to answer "what does the code say here" and to discover the amendments that override the model book
- https://www.cslb.ca.gov/Resources/GuidesAndPamphlets/OwnerBuilderOverview.aspx - California CSLB owner-builder overview: the exemption, the declaration, and the liability you accept by pulling your own permit; the honest framing for the DIY questions
- https://www.energy.ca.gov/programs-and-topics/programs/building-energy-efficiency-standards - Title 24, California's energy code: compliance manuals and documents; the state layer the IRC-based sources don't cover

## 5. The data - the corpus survey starts here

Nearly everything here is free and authority-published, and each source doubles
as corpus material and as the model's citation targets. Check each site's terms
before training on anything; the code-text issue is noted under
Public.Resource.Org below.

- https://www.govinfo.gov/content/pkg/GOVPUB-HS5_100-PURL-gpo15862/pdf/GOVPUB-HS5_100-PURL-gpo15862.pdf - FEMA P-499, Home Builder's Guide to Coastal Construction: fact-sheet builder guidance on coastal high wind and flood, clean public-domain PDF text, and directly relevant to a Monterey-area build
- https://awc.org/publications/ - American Wood Council's free publications: joist and rafter span tables and design guides including deck construction; the numbers behind most framing questions
- https://www.apawood.org/publications - APA's free engineered-wood design guides: sheathing, panel spans, and detail drawings that pair with AWC's tables
- https://www.epa.gov/radon/radon-resistant-construction-basics-and-techniques - EPA's radon-resistant new construction techniques: cheap during the pour, expensive afterward; a canonical "nobody told the newbie" topic
- https://www.nachi.org/sop.htm - InterNACHI's Standards of Practice for home inspections: what an inspector is required to check; a ready-made evaluation checklist and a corpus of "what gets flagged"
- https://public.resource.org/ - Public.Resource.Org and the fight over public access to codes and standards: read before training on code text, because ICC's free viewer is viewing-only and its terms restrict reproduction

## 6. Fine-tuning for the explainer's voice - not the facts

- https://arxiv.org/abs/2405.09673 - LoRA Learns Less and Forgets Less - the evidence behind the split this guide expects: adapters pick up tone, format, and answer structure while leaving knowledge to retrieval

The rest of the fine-tune inputs are generic: preference pairs built from real
newbie questions and the answers the author wishes they'd gotten (distilabel
and Argilla, both in the root references.md), and the DPO trainer, also there.
One format-specific idea worth stealing: answers should end with the "check
with your building department / an engineer" handoff, so put that behavior in
the training data from the first batch.

## 7. Evaluation - safety-critical, open-ended, and you are building the rubric

There is no public benchmark for construction-advice quality, so the eval is
the genuinely new work plan.md flags. The human half: a contractor or inspector
grades blind answers (the JAMA template in section 1). The scriptable half:

- https://docs.ragas.io/en/stable/ - RAGAS: faithfulness and answer-relevancy metrics; faithfulness here means "every claim traces to a retrieved code section or publication"
- https://arxiv.org/abs/2306.05685 - Judging LLM-as-a-Judge with MT-Bench and Chatbot Arena: the canonical judge paper; use it, but ground the judge in the retrieved citations and trust it less than the human expert
- https://arxiv.org/abs/2207.05221 - Language Models (Mostly) Know What They Know: calibration and selective prediction; "go ask the building department" must be a first-class answer, and this is the research behind knowing when
- https://arxiv.org/abs/2411.08320 - Responsible AI in Construction Safety: LLMs applied to the sector where the fatality statistics live; frames exactly the stakes the eval has to catch

## Not verified to the curl-200 standard

All six have useful content but block automated checks; read them in a browser.

- https://codes.iccsafe.org/codes/i-codes - ICC's official free read-only viewer for the IRC and its sibling codes; bot-blocks curl (403)
- https://www.huduser.gov/portal/publications/Residential-Structural-Design-Guide-Second-Edition.html - HUD's Residential Structural Design Guide, 2nd edition, the single best free structural primer for a newbie (loads, framing, connections), free PDF; returns 202 to curl
- https://basc.pnnl.gov/ - DOE's Building America Solution Center: per-assembly guides and images, a strong corpus candidate; rejects plain curl (400)
- https://www.osha.gov/residential-fall-protection - OSHA's residential fall-protection guidance; if the newbie does any of the work themselves, falls are the hazard that kills; bot-blocks curl (403)
- https://www.dgs.ca.gov/BSC/Codes.aspx - California Building Standards Commission, publisher of Title 24 including the CRC; bot-blocks curl (403)
- https://www.co.monterey.ca.us/government/departments-i-z/resource-management-agency/building-services - the county building department: permits, plan check, inspections; find the actual build's equivalent, it is the most important site in this list; bot-blocks curl (403)

## Suggested reading order

1. Ovadia et al., plus the root "Do you even need to fine-tune?" page - settle the RAG-first decision before anything else
2. Section 5 - survey the free corpus, starting with HUD's design guide and the BSC library
3. The RAG build: cookbook recipe, then chunking, hybrid search, and the citation engine
4. The field: PGH, GBA, and BSC digests - learn the building science you will grade answers against
5. UpCodes and the CSLB owner-builder page - the jurisdiction layer and the liability line
6. RAGAS, the JAMA eval template, and the calibration paper - then the voice fine-tune last

The corpus is the project here: nearly everything above is free and
authority-published, which is also what makes the RAG-first answer easy.
