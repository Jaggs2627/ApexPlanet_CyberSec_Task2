Step 1: Passive Reconnaissance
Define and explain how a security analyst uses the following open-source intelligence (OSINT) tools without interacting with the target network:
WHOIS Lookup: Explain that this queries public registries to find domain ownership, registration dates, name servers, and administrative contact emails.
NSLookup: Explain that this queries DNS servers to resolve domain names into IP addresses and uncover specific records (like MX records for mail servers).
Google Dorking: Provide examples of advanced search operators used to find leaked configurations or sensitive files (e.g., site:target.com filetype:sql).
Shodan: Explain that this is a search engine for internet-connected devices that gathers banner data from public-facing ports, mapping out vulnerable infrastructure without hitting them directly.
Part B: Active Reconnaissance
Explain the conceptual difference when you transition from passive gathering to active targeting:
Ping Sweep: Document how an analyst sends ICMP Echo Request packets across an entire subnet range (e.g., 10.0.2.1 to 10.0.2.254) to see which host machines are online and responding.
Banner Grabbing: Document how an analyst establishes a basic connection to an open port (like Port 21 or 80) to read the text string sent back by the service, exposing the exact software name and version number.
