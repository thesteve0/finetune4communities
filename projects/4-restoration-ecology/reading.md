# Reading list: tuning a model for restoration ecology in the southern Santa Lucia Range

Compiled 2026-11-11. All URLs verified with curl (HTTP 200) on this date, except
the three in the last section, which block automated checks.

Aimed at someone who has already done the text-SFT pass of example 1. The root
references.md covers the generic toolchain (TRL/PEFT, LoRA, MLflow, the eval
guidebook, Argilla/distilabel, serving), and the fine-tuning mechanics here are
the same text stack, so none of that is repeated. This list adds what is
specific to this project: the RAG-versus-fine-tuning decision for a factual
domain expert, the restoration ecology itself, the hyper-local sources that make
the model "expert for this place", the data-availability survey plan.md calls
for, and evaluation that needs domain expertise.

plan.md's framing is the spine: RAG is likely the primary pattern (facts about
species, sites, and treatments do not belong in weights), with fine-tuning as a
complement for the advisor's voice and format - and possibly LoRA-based
continued pretraining over curated local text. The data section below doubles
as the start of that survey. Scope assumption: "southern Santa Lucia Range"
means the Big Sur coast and backcountry (Ventana and Silver Peak Wildernesses,
Monterey Ranger District of Los Padres NF), Carmel Valley and the Santa Lucia
Preserve, with the range's edges - Monterey pine/cypress country on the coast,
Hastings in upper Carmel Valley - included where they illuminate the whole.

## 1. The honest framing - what kind of "expert" is this model

The "Do you even need to fine-tune?" logic (the IBM piece in references.md)
applies most strongly here of all five examples. These sources sharpen the
decision with evidence rather than opinion:

- https://arxiv.org/abs/2312.05934 - Ovadia et al., Fine-Tuning or Retrieval? - the head-to-head of knowledge injection into small models; finding: retrieval consistently beats fine-tuning for factual recall, which is the default assumption this project should have to argue against
- https://arxiv.org/abs/2004.10964 - Gururangan et al., Don't Stop Pretraining - the canonical argument that domain text, even scarce domain text, is worth adapting to; under the repo's no-full-fine-tuning rule this means continued pretraining as LoRA, not from-scratch DAPT
- https://arxiv.org/abs/2405.09673 - LoRA Learns Less and Forgets Less - the evidence behind the split this guide expects: adapters learn format and tone while leaving knowledge to retrieval

## 2. RAG - the likely primary pattern

- https://arxiv.org/abs/2005.11401 - Lewis et al., the original RAG paper; the formulation every later "retrieval-grounded expert" system inherits
- https://huggingface.co/learn/cookbook/en/rag_zephyr_langchain - HF Cookbook: a small open model plus a local retriever, runnable end-to-end; the closest recipe to what this example ships
- https://www.pinecone.io/learn/chunking-strategies/ - chunking strategies; the quality lever that matters most on this corpus of heterogeneous agency reports, theses, and plans
- https://www.anthropic.com/news/contextual-retrieval - contextual retrieval: prepend chunk context before embedding; a stack-agnostic technique that helps most on technical corpora where chunks are meaningless alone
- https://docs.llamaindex.ai/en/stable/examples/query_engine/citation_query_engine/ - LlamaIndex's citation query engine: answers that quote their sources back, the shape a restoration advisor's answers need to take
- https://sbert.net/ - sentence-transformers docs: open, local embeddings for the retriever

## 3. The field - restoration ecology fundamentals

- https://semspub.epa.gov/work/01/554371.pdf - the SER International Primer on Ecological Restoration (PDF mirror; ser.org itself blocks automated checks) - definitions, the reference-ecosystem concept, and project standards; the vocabulary the model has to speak
- https://research.fs.usda.gov/treesearch/61909 - Postfire restoration framework for national forests in California (USFS) - the federal post-fire recovery logic that applies directly to Los Padres landscapes after Basin, Soberanes, and Dolan
- https://www.ucpress.edu/books/introduction-to-california-plant-life/paper - Ornduff, Faber, and Keeler-Wolf's Introduction to California Plant Life - the short vegetation primer behind Jepson and the MCV; the background a generalist reader needs before the hyper-local material
- https://californiaoaks.org/ - California Oaks - oak woodland conservation and restoration, for the ecosystem that dominates the Carmel Valley foothills and most private-land restoration in the range

## 4. The place - the southern Santa Lucia Range

- https://slconservancy.org/ - Santa Lucia Conservancy - 20,000-acre private preserve in the Santa Lucias with an active research and restoration program; the closest institutional analog to what this model would advise on
- https://slconservancy.org/explore/ - the Conservancy's resource library: publications, peer-reviewed articles, and species guides, all about this exact landscape
- https://ucnrs.org/reserves/landels-hill-big-creek-reserve/ - Landels-Hill Big Creek Reserve (UC Natural Reserve System, Big Sur coast) - decades of research and monitoring on a protected Big Sur watershed
- https://hastingsreserve.org/ - Hastings Natural History Reservation (UC Berkeley, upper Carmel Valley) - long-term ecological research in the range's inland foothills
- https://www.fs.usda.gov/r05/lospadres - Los Padres National Forest - the Monterey Ranger District manages most of the range's public land, so its plans and programs bound what public-land restoration can happen
- https://www.fs.usda.gov/database/feis/plants/tree/abibra/all.html - FEIS literature review of Santa Lucia fir (Abies bracteata), the range's signature endemic; FEIS species reviews are also a ready-made literature base for many more local species
- https://www.fs.usda.gov/database/feis/plants/tree/pinrad/all.html - FEIS review of Monterey pine, at the range's coastal edge, where one of the world's rarest pines meets heavy development pressure
- https://www.ventanaws.org/ - Ventana Wildlife Society - condor reintroduction and habitat work in the Big Sur backcountry; the region's most visible restoration story
- https://www.suddenoakdeath.org/ - California Oak Mortality Task Force - sudden oak death, active in Monterey County; the disease dimension any oak restoration here must plan around
- https://www.sanclementedamremoval.org/ - San Clemente Dam removal and Carmel River reroute - the region's flagship river-restoration project and a case study in multi-agency community restoration
- https://bigsurlandtrust.org/ - Big Sur Land Trust - conservation and stewardship on the private-land side of the coast

## 5. The data - the availability survey starts here

plan.md requires a data-availability survey before committing to this example.
These are the sources to survey; several are also dataset generators, not just
reading material:

- https://www.calflora.org/entry/wgh.html - Calflora's What Grows Here - species lists for any polygon; the natural source for place-conditional synthetic data
- https://www.cal-ipc.org/plants/inventory/ - the Cal-IPC invasive plant inventory - the threat side of restoration, with ratings per species
- https://www.rareplants.cnps.org/ - CNPS Inventory of Rare and Endangered Plants - the rare-species constraints any project must respect
- https://vegetation.cnps.org/ - the Manual of California Vegetation - the classification vocabulary (maritime chaparral, redwood, oak woodland...) in which local vegetation gets described
- https://ucjeps.berkeley.edu/eflora/ - the Jepson eFlora - the botanical authority the model's species claims should agree with
- https://www.wildlife.ca.gov/Data/CNDDB - CNDDB - California's official rare-species occurrence database
- https://www.wildlife.ca.gov/Data/VegCAMP - CDFW's VegCAMP - state vegetation mapping and classification, with datasets downloadable
- https://ucjeps.berkeley.edu/consortium/ - the Consortium of California Herbaria - vouchered occurrence records; the ground truth when Calflora or iNaturalist disagrees
- https://api.inaturalist.org/v1/docs/ - the iNaturalist API - community-science observations queryable by place and taxon; see the repo's ethics page before using it in training data
- https://data.ca.gov/dataset/california-fire-perimeters-all - CAL FIRE FRAP fire perimeters - the fire history (Marble-Cone 1977, Basin 2008, Soberanes 2016, Dolan 2020) that structures post-fire restoration here
- https://landfire.gov/ - LANDFIRE - national vegetation, fuels, and fire-regime layers for the range
- https://websoilsurvey.nrcs.usda.gov/app/ - NRCS Web Soil Survey - soils, the substrate of every restoration plan
- https://digitalcommons.calpoly.edu/theses/ - Cal Poly's thesis repository - hyper-local Central Coast ecology studies that never reached journals; the scarcest, most valuable texts for the corpus

## 6. Fine-tuning for the advisor's voice - not the facts

- https://docs.unsloth.ai/basics/continued-pretraining - continued pretraining over curated local text with LoRA; the recipe if the model needs more of the corpus in its weights (CUDA-only, per the README's hardware caveat)

## 7. Evaluation - domain expertise is the metric

The eval question here is "is it right for this place", which only a domain
expert can grade - the part plan.md flags as genuinely new:

- https://pmc.ncbi.nlm.nih.gov/articles/PMC12080840/ - PLOS One: carefully designed LLM pipelines achieve expert-level retrieval from the Conservation Evidence database - both an architecture to copy (retrieval-grounded answers over curated evidence) and an evaluation template for asking whether the model matches expert knowledge
- https://docs.ragas.io/en/stable/ - RAGAS - faithfulness and answer-relevancy metrics for a citation-grounded assistant, the scriptable half of the eval before the human expert review

## Not verified to the curl-200 standard

All three have useful content but block automated checks; read them in a browser.

- https://www.ventanawild.org/ - Ventana Wilderness Alliance - trail stewardship, weed pulls, and restoration reports from inside the Ventana Wilderness; rate-limits curl (403)
- https://www.sciencedirect.com/science/article/pii/S1574954126001056 - 2026 assessment of how much ecological knowledge general LLMs hold (species presence, ranges, traits) - bot-blocks curl (403); the baseline-measurement study for what a general model already knows
- https://conbio.onlinelibrary.wiley.com/doi/10.1111/cobi.70287 - Conservation Biology review of LLM applications and recommended practices in ecology and conservation - bot-blocks curl (403)

## Suggested reading order

1. SER primer, then Introduction to California Plant Life - speak the vocabulary
2. Ovadia et al. and the USFS postfire framework - the decision and the regional shape of the work
3. Section 5, starting with Calflora and Cal-IPC - the availability survey, before committing
4. The RAG build: cookbook recipe, then chunking and the citation engine
5. Continued pretraining (if at all) and the RAGAS + Conservation Evidence evals

The Santa Lucia Conservancy's resource library is the single best place to see
what "expert for this range" already looks like in practice.
