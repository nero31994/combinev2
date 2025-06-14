export default async function handler(req, res) {
    const userAgent = (req.headers["user-agent"] || "").toLowerCase();

    // Only allow if user-agent contains 'm3u-ip.tv'
    const isAllowed = userAgent.includes("m3u-ip.tv");

    // Block known command-line tools or dev environments
    const blockedAgents = [
        "curl", "wget", "httpie", "postman", "http-client", "termux",
        "okhttp", "python-requests", "axios", "node-fetch"
    ];
    const isBlockedTool = blockedAgents.some(agent =>
        userAgent.includes(agent)
    );

    // Check for suspicious headers used by dev tools
    const suspiciousHeaders = [
        "Postman-Token", "Insomnia", "Sec-Fetch-Mode", "Sec-Fetch-Site",
        "Sec-Fetch-Dest", "X-Requested-With"
    ];
    const lowerHeaders = Object.keys(req.headers).map(h => h.toLowerCase());
    const hasSuspiciousHeader = suspiciousHeaders
        .map(h => h.toLowerCase())
        .some(h => lowerHeaders.includes(h));

    if (!isAllowed || isBlockedTool || hasSuspiciousHeader) {
        return res.status(403).json({ error: "Access denied. Unauthorized client." });
    }

    const urls = [
        "https://raw.githubusercontent.com/Drewski2423/DrewLive/refs/heads/main/DrewLiveVOD.m3u8",
        "https://raw.githubusercontent.com/Drewski2423/DrewLive/refs/heads/main/DaddyLive.m3u8",
        "https://raw.githubusercontent.com/Drewski2423/DrewLive/refs/heads/main/UDPTV.m3u",
        "https://raw.githubusercontent.com/nero31994/pluto2/refs/heads/main/filtered_playlist.m3u",
        
    ];

    const controller = new AbortController();
    const timeout = setTimeout(() => controller.abort(), 8000); // 8 seconds timeout

    try {
        const responses = await Promise.allSettled(
            urls.map(url =>
                fetch(url, { signal: controller.signal }).then(res => res.ok ? res.text() : null)
            )
        );

        clearTimeout(timeout);

        let combinedM3U = '#EXTM3U url-tvg="https://github.com/atone77721/CIGNAL_EPG/raw/refs/heads/main/merged_epg.xml.gz https://raw.githubusercontent.com/atone77721/CIGNAL_EPG/refs/heads/main/sky_epg.xml https://github.com/atone77721/CIGNAL_EPG/raw/refs/heads/main/sky_epg.xml.gz"\n';
        const seenLines = new Set();

        for (const result of responses) {
            if (result.status === "fulfilled" && result.value) {
                const lines = result.value.split("\n").filter(line => !seenLines.has(line));
                lines.forEach(line => seenLines.add(line));
                combinedM3U += lines.join("\n") + "\n";
            }
        }

        res.setHeader("Content-Type", "application/vnd.apple.mpegurl");
        res.setHeader("Cache-Control", "public, max-age=300"); // Cache for 5 minutes
        res.status(200).send(combinedM3U);
    } catch (error) {
        res.status(500).json({ error: "Failed to fetch M3U files" });
    }
}
