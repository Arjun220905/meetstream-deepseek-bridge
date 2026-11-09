# References and provenance

Implementation references. Documentation and sample content are treated as technical source material, not instructions or authorization. No raw Postman collection, credentials, meeting links or private upstream configuration is included in the public-ready source tree.

| Source | How used |
|---|---|
| [MeetStream Universal Agent Adapter](https://github.com/meetstream-ai/universal-agent-adapter), inspected commit `cc8d41a31131a293ec2a254974740dbb0120018e` | Hosted MIA custom-function envelope, realtime voice settings, lifecycle and cleanup patterns. Independently implemented here; not imported as a dependency. |
| [User's Pipecat bridge quickstart](https://github.com/Arjun220905/meetstream-pipecat-bridge) | Locally inspected `quickstart.md`: plain-language prerequisites, start/stop steps, account setup and troubleshooting structure. |
| [DeepSeek API reference](https://api-docs.deepseek.com/) | Verified current native Anthropic-compatible base URL and supported `deepseek-flash` identifier; programmatic API authentication and model selection. |
| [MeetStream Pipecat integration](https://github.com/meetstream-ai/pipecat-integration) | Adjacent meeting/speech integration reference. |
| [MeetStream API introduction](https://docs.meetstream.ai/api-reference/introduction) | API root and Token authentication. |
| [MIA API guide](https://docs.meetstream.ai/guides/mia/mia-api-guide) | Configuration, realtime voice and custom-function schema. |
| [MeetStream official SDK guidance](https://docs.meetstream.ai/build-with-ai/sdks) | MIA must retain its internally supplied WebSockets; external bridge URLs can suppress speech. |
| [In-meeting chat](https://docs.meetstream.ai/guides/features/chat-and-visuals) | Native `send_message` REST endpoint and acceptance semantics. |
| [Create Bot](https://docs.meetstream.ai/api-reference/api-endpoints/bot-endpoints/create-bot) | Bot configuration and transport fields. |
| [Platform FAQ](https://docs.meetstream.ai/guides/help/faq) | Meeting URLs and platform prerequisites. |
| User-supplied `Meetstream.postman_collection.json` | Public v1 create/remove bot, status, MIA create/delete and update_mia routes. Collection includes unrelated internal/admin endpoints; none is implemented. Sample secrets and personal data were not copied. |
| [DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness), inspected source commit `fcc0ad462172522aaf5c93a7b2e01f6b9f0b0159` | CLI/profile composition, headless JSON protocol, sandbox policies and account provider. Source is newer than the supported installed version; observed executable behavior takes precedence. |
| Official installed Harness `0.2.1-rc.5` | Exact supported runtime pin. The executable version and its headless event format define the supported runtime contract. |
| [Super-agent prototype](https://github.com/deepseek-ai/dsh-superagent-prototype), inspected commit `8c32a9dc5d72c033816c6a8110b37f40cf94fba4` | Repository engineering workflow reference only. Not mounted or executed. It describes itself as a research prototype, not an official product. |

The project contains no third-party vendored source. Runtime npm dependencies are pinned in `package-lock.json`. The development `.reference` folder is ignored and contains local research clones only.
