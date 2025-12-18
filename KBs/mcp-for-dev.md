# 6 Open-Source MCP Servers Every Dev Should Try

[![Algo Insights](https://miro.medium.com/v2/resize:fill:64:64/1*kcZSvDRaxmbCAUerJnPTUw.png)](https://medium.com/@algoinsights?source=post_page---byline--b3cc6cf6a714---------------------------------------)

[Algo Insights](https://medium.com/@algoinsights?source=post_page---byline--b3cc6cf6a714---------------------------------------)

**Follow**

5 min read**·**

Jul 2, 2025

617

**5**

I’ve tested over 100 MCP servers in the past few months.

And let me tell you, some of them are absolute gold.

MCP, if you’re new to it, is this open-source protocol from Anthropic that lets AI like Claude chat with data, APIs, or tools without breaking a sweat.

I’m sharing my top six picks.

These are free, open-source, and perfect for making your AI projects pop.

## 1.** ** =Graphiti= : AI That Remembers Stuff

I was so done with my AI forgetting everything after a task. Like, come on, I need it to recall what we worked on last week!** ****Graphiti MCP Server** fixes that. It builds these neat knowledge graphs in Neo4j, so your AI can track relationships and history. Think of it as sticky notes for your bot.

* Lets your AI look back in time (e.g., “What changed in my project last month?”).
* Uses Neo4j, so it handles big data like a champ.
* Sets up faster than my morning coffee.

Here’s what I did to get it running:

```
# Grab the CLI
pipx install 'rawr-mcp-graphiti[cli]'

# Fire up Docker and configs
cd my-cool-project
graphiti compose
graphiti up -d
# Start a graph
graphiti init team-tracker
```

Then, in Cursor (my go-to MCP client), I asked:

```
query = "Who joined the team this week?"
response = mcp_client.query("http://localhost:8000/sse", query)
print(response)  # Spits out: ["Bob joined on June 25"]
```

 **Link** :** **[Graphiti MCP Server](https://github.com/getzep/graphiti/tree/main/mcp_server)

![](https://miro.medium.com/v2/resize:fit:978/1*Sb8RMIqWT4L653c3I0Uhew.png)

## 2. Opik: Keep an Eye on Your AI

Last month, my chatbot was acting sluggish, and I had no clue why.** ****Opik MCP Server** from Comet saved the day. It’s like a spy tool for your AI, tracking what it’s doing and giving you stats. Plus, it’s free and open-source.

* Shows you what’s up with your AI’s performance.
* Tracks stuff like how fast it responds.
* Easy to tweak for your setup.

Here’s how I got it working:

```
# Clone it
git clone https://github.com/comet-ml/opik-mcp
cd opik-mcp
pip install -r requirements.txt
python server.py
```

Then, I checked my app’s stats:

```
query = "How’s my chatbot doing?"
response = mcp_client.query("http://localhost:8080/sse", query)
print(response)  # Shows: { "app": "chatbot", "response_time": "0.3s" }
```

 **Link** :** **[Opik MCP Server](https://github.com/comet-ml/opik-mcp)

![](https://miro.medium.com/v2/resize:fit:922/1*k-XwkkyOIMYczSwn4rfDzA.png)

## 3. Ragie: Ask Your Videos Questions

I needed to dig through a bunch of meeting videos for a project, and** ****Ragie MCP Server** was a lifesaver. It does this thing called multimodal RAG, so you can upload a video, ask stuff, and get answers with exact timestamps. It’s wild.

* Pinpoints moments in videos (e.g., “What’s at 3 minutes?”).
* Works with videos, docs, whatever.
* Hooks up with Claude super easy.

Here’s what I did:

```
# Clone and run
git clone https://github.com/ragieai/ragie-mcp-server
cd ragie-mcp-server
npm install
npm start
```

Upload a video and ask away:

```
// Upload
fetch('http://localhost:3000/ingest', {
  method: 'POST',
  body: new FormData().append('video', 'team-meeting.mp4')
});

// Ask a question
const mcpClient = require('mcp-client');
mcpClient.query('http://localhost:3000/sse', 'What's at 4:00?', (res) => {
  console.log(res);  // Says: "At 4:10, we talked about budgets."
});
```

 **Link** :** **[Ragie MCP Server](https://github.com/ragieai/ragie-mcp-server)

**Press enter or click to view image in full size**![](https://miro.medium.com/v2/resize:fit:1400/1*qilmdU3qbXjorel611uaVQ.png)

## 4. Bright Data: Scrape Without the Stress

Scraping websites is the worst — blocks, CAPTCHAs, bleh.** ****Bright Data MCP Server** makes it painless. It’s got 30+ tools and picks the best one for the site you’re hitting. I used it to grab prices for a side gig, and it didn’t flinch.

* Dodges website blocks like a ninja.
* Figures out the best tool for the job.
* Totally free to use.

Here’s the setup:

```
# Clone and start
git clone https://github.com/luminati-io/brightdata-mcp
cd brightdata-mcp
npm install
npm start
```

Scrape some data:

```
const mcpClient = require('mcp-client');
mcpClient.query('http://localhost:4000/sse', 'Grab prices from gadgetshop.com', (res) => {
  console.log(res);  // Lists: [{ "item": "Phone", "price": "$599" }, ...]
});
```

 **Link** :** **[Bright Data MCP Server](https://github.com/luminati-io/brightdata-mcp)

**Press enter or click to view image in full size**![](https://miro.medium.com/v2/resize:fit:1400/1*h2VRYA-H23nl-mtjanxd9w.png)

## 5. Jupyter: Run Notebooks with AI

I’m a sucker for Jupyter notebooks — great for quick data experiments.** ****Jupyter MCP Server** lets me control them with Claude, so I can write code or add notes without clicking around. It’s a total time-saver.

* Run code cells with plain English.
* Perfect for data nerds like me.
* Free and open to mess with.

Here’s how I set it up:

```
# Clone and run
git clone https://github.com/datalayer/jupyter-mcp-server
cd jupyter-mcp-server
pip install -r requirements.txt
python server.py
```

Run a quick plot:

```
query = "Make a line plot of [1, 3, 2]"
response = mcp_client.query("http://localhost:8888/sse", query)
print(response)  # Runs: plt.plot([1, 3, 2])
```

 **Link** :** **[Jupyter MCP Server](https://github.com/datalayer/jupyter-mcp-server)

**Press enter or click to view image in full size**![](https://miro.medium.com/v2/resize:fit:1400/1*lM4IHjTUhlIHuwtIzW4Amw.png)

## 6.** **=MindsDB: All Your Data in One Place=

I was juggling Slack, a database, and Gmail for a dashboard project, and** ****MindsDB MCP Server** made it stupidly easy. It hooks up to 200+ platforms, so you can query anything — SQL or plain words. It’s got a huge community, too.

* Connects to everything — databases, apps, you name it.
* Ask stuff like, “What’s in my inbox?” and it delivers.
* Free and backed by 28K+ GitHub stars.

Spin it up with Docker:

```
docker run -p 47334:47334 mindsdb/mindsdb
```

Query your Slack:

```
query = "What’s new in #coding?"
response = mcp_client.query("http://localhost:47334/sse", query)
print(response)  # Shows: ["Message: 'Check this bug fix!'"]
```

 **Link** :** **[MindsDB MCP Server](https://github.com/mindsdb/mindsdb)

**Press enter or click to view image in full size**![](https://miro.medium.com/v2/resize:fit:1400/1*PPU74LtgVIUZngemviJ-3A.png)

Image by — mindsdb

I’ve built all sorts of stuff — apps, bots, dashboards — and MCP servers cut out so much grunt work.

They let your AI talk to tools without you writing endless glue code. Graphiti keeps things organized, Bright Data tackles the web, and MindsDB handles all your data. Pick one, clone it, and play around. Most take under 10 minutes to set up.
