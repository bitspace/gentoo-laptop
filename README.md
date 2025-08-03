# LLM-Guided Laptop Build Experiment

## Purpose and Background

The primary goal of this project is to gain exposure, experience, and maybe some muscle memory in the realm of large language models, especially with the CLI-based agentic model orchestrator tools: Claude Code, Gemini CLI, and OpenAI Codex. I may retroactively add GitHub Copilot into this, but that's something of a different beast, being a preschooler-targeted "click the pictures and make it go" type of product. (My disdain for the WIMP paradigm is legendary.)  

The overall goal is to end up with a laptop that suits some of my computing needs (gaming, coding, tinkering with technology, AI, and maybe some music production). I have a System76 Kudu laptop that I'm not really using; it was most recently running Windows 11 to facilitate gaming. However, I couldn't tolerate Wintendo for very long and rage-wiped the disk. Currently it has a barebones Gentoo Linux installation after some iterations on an earlier variant of this experiment to guide the installation. Gentoo turned out not to be a great choice because of the ridiculous source compile times of many core system packages (gcc, llvm, kernel, other suff). Claude Code could not get past the hour+ compile times without timing out and crashing. It turns out that the "build everything from source" is not compatible with "rapid iteration" style work, which is where the conversational nature of the LLM chatbots and agentic tools really shines.  

Back to the drawing board, I decided to use this LLM-consensus approach to help me decide on the "best" Linux distribution and the "best" window manager and environment to use, optimizing for how easily the system configuration and other data can be parsed by a LLM.  

I hadn't even considered NixOS until Gemini threw me that outside-the-box curve ball (and I didn't even change any model parameters like temperature!) making the case for the declarative immutable system configuration paradigm being extremely LLM-friendly and automation-friendly.  

As soon as I modified my general LLM prompt and my system prompt "Linux expert" (Tinus Lorvalds is his name) to consider NixOS alongside Arch, Gentoo, and other more "traditional" distributions, NixOS became a top contender with almost all models.  

I have held for decades that the way we interact with computers is twisted and unnatural. Typing is not a natural human skill. Information presented on a screen in a very rough "desktop" metaphor is also extremely limiting and unnatural: hence my disdain for the WIMP (Windows, Icons, Menu, Pointer) paradigm. The most "natural" human information exchange channels are voice/hearing and writing (with a handwriting implement, not a typewriter)/reading.  

The explosion of NLP and LLM's (as well as speech recognition technology such as Whisper) in the past couple of years has changed things dramatically. While I'm still typing, I can now begin to integrate "natural language" or "plain English" in my typed commands and inquiries, rather than having to memorize a ton of archaic text things. With some additional work, rather than typing, I can speak; this is fairly commonplace with smart phones and increasingly with tablet and laptop form factors.  

## The Goal

Using LLM agent technology as much as possible (use plain English to issue commands), build a customized laptop that is optimized for automation/scriptability and LLM parsability. The system must be usable as a daily driver for software development, AI/ML/LLM/MLOps work, gaming (Steam/WINE/Proton), and possibly music production.  

Prompt several different LLM's with the same goal and guidelines, gather their output, and have my agent orchestrator synthesize a "most suitable" plan and configuration from a consensus of the other models, providing me the opportunity to override any decision.  

## The Tools

I've been working a lot with [Claude Code](https://www.anthropic.com/claude-code) in my freelance/personal/pet project space. It is extremely powerful, and is the best demonstration of what I think is going to be how software is going to be developed from now on. My day job limits which AI models we have access to (none, outside of whatever they make available through the corporate Azure AI contract), has involved more use of GitHub Copilot, increasingly in Agent mode. I've also tinkered a little bit with [Gemini CLI](https://github.com/google-gemini/gemini-cli), and for fairness, I also want to assess OpenAI's contribution to the space: [Codex CLI](https://help.openai.com/en/articles/11096431-openai-codex-cli-getting-started).  

At the same time, I am learning how the top LLM's (at the time of this writing, 03 August 2025) behave with different inputs. A few of them provide dedicated chat interfaces, while others were accessed through a ML model gateway provider. I generated a "Linux Expert" system prompt (one that I've been using for many months for Linux expertise), documented the target system's hardware specifications, and wrote a prompt to guide the model to choose the top 5 Linux distributions and the top 5 graphical environments (window managers or desktop environments) given my parameters, and to provide a high-level implementation plan. I then instructed the agentic orchestrator (Claude Code, Gemini CLI, Codex CLI) to analyze and synthesize these into a "best of" recommendation and a more detailed implementation plan. Each of these is in its own branch in this repository.

### The Models

These are the models I used to contribute to the consensus. The selection was not arbitrary, but came from a general sense of the "best" models in common use at this time.  

- [ChatGPT-o3](llm-responses/chatgpt-o3-deepresearch.md) - "Deep Research" enabled. I used the consumer-facing ChatGPT UI, not the playground, so I did not have an opportunity to provide a system prompt. I entered the text of [my system prompt](llm-inputs/linux-sme-system-prompt.md) as the start of the user prompt, then input the content of [the user prompt](llm-inputs/initial-llm-prompt.md), and attached [the system specifications](llm-inputs/gimli-system-specifications.md).
- [Claude Opus 4](llm-responses/claude-opus-4-research.md) - Enabled "Research" mode. I did not have an opportunity to provide a system prompt. I entered the text of my system prompt as the start of the user prompt, then input the content of the user prompt, and attached the system specifications.
- [Cohere Command-A](llm-responses/cohere-command-a-03-2025.md) - Cohere's playground provides an input field for a system prompt/message, where I input the text of my system prompt. The UI does not provide a facility for attaching files, so I pasted the contents of the system specifications at the start of the user prompt, and then pasted the contents of the actual prompt.
- [Deepseek R1 05/28](llm-responses/deepseek-r1-0528.md) - My interface to this model was via [Fireworks AI](https://app.fireworks.ai) which is somewhat limited, even in the Playground. The Fireworks UI does not provide either a system prompt or attachment capability, so I [created a user prompt](llm-inputs/generic-chatbot-input.md) that simulates the system prompt and includes the system specifications and the prompt text.
- [Gemini 2.5 Pro](llm-responses/gemini-2.5-pro-deep-research.md) - "Deep Research" mode enabled. While the Gemini UI does provide a facility for activating system prompts (they call it "Gems") I did not use that for this project because it disables access to Deep Research mode. Instead, I attached the target system specifications, entered the system prompt as the first part of the user prompt, and then entered the content of the prompt.
- [Kimi K2 Instruct](llm-responses/kimi-k2-instruct.md) - via the Fireworks.AI playground. Same caveats as described with the Deepseek model.
- [Llama 3.1 405B Instruct](llm-responses/llama.md) - via the Fireworks.AI playground. Same caveats as previously described models used via that interface.
- [Mistral Le Chat](llm-responses/mistral.md) - via [Mistral's chat UI](https://chat.mistral.ai/chat) with "Research" activated. Activating "Research" disables the file attachment ability, so I used the [generic chatbot input](llm-inputs/generic-chatbot-input.md) that collates system prompt, target system specifications, and user prompt into one user prompt.
- [Perplexity](llm-responses/perplexity-research.md) - via [the Perplexity web UI](https://www.perplexity.ai/); enabled "Research" mode, attached target system specifications, entered "system prompt" as first portion of user prompt.
- [Qwen3 Coder 480B A35B Instruct](llm-responses/qwen.md) - via Fireworks.AI playground. Used generic chatbot combined prompt.
