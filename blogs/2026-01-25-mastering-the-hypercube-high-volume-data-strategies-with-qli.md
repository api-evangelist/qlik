---
title: "Mastering the HyperCube: High-Volume Data Strategies with @qlik/api"
url: "https://community.qlik.com/t5/Design/Mastering-the-HyperCube-High-Volume-Data-Strategies-with-qlik/ba-p/2540100"
date: "Sun, 25 Jan 2026 15:14:38 GMT"
author: "Ouadie"
feed_url: "https://community.qlik.com:443/cyjdu72974/rss/board?board.id=qlik-design-blog"
---
<p>If you have been building custom web applications or mashups with Qlik Cloud, you have likely hit the "10K cells ceiling" when using Hypercubes to fetch data from Qlik.<br /><font color="#808080"><em>(Read my previous posts about Hypercubes <a href="https://community.qlik.com/t5/Design/Interacting-with-data-using-Enigma-js-Hypercubes-and-List/ba-p/1970913" target="_self">here</a>&nbsp;and&nbsp;<a href="https://community.qlik.com/t5/Design/Interacting-with-data-using-Enigma-js-PT-2-Creating-Filters-with/ba-p/1987764" target="_self">here</a>)</em></font></p>
<p>You build a data-driven component, it works perfectly with low-volume test data, and then you connect it to production; and now suddenly, your list of 50,000+ customers cuts off halfway, or your export results look incomplete.</p>
<p>This happens because the Qlik Engine imposes a strict limit on data retrieval: <strong><a href="https://qlik.dev/extend/build-extension/hypercube/" target="_self">a maximum of 10,000 cells per request</a>.</strong> If you fetch 4 columns, you only get 2,500 rows (4 <font size="2"><em>(columns)</em></font> x 2500 = 10,000 <em><font size="2">(max cells)</font></em>).</p>
<p>In this post, I’ll show you how to master high-volume data retrieval using the two strategies: <strong>Bulk Ingest</strong> and <strong>On-Demand Paging</strong>, using&nbsp;the&nbsp;<a href="https://qlik.dev/toolkits/qlik-api/" target="_self">@qlik/api</a> library.</p>
<h5><font color="#008000"><strong>What is the 10k Limit and Why Does It Matter?</strong></font></h5>
<p>The Qlik Associative Engine is built for speed and can handle billions of rows in memory. However, transferring that much data to a web browser in one go would be inefficient. To protect both the server and the client-side experience, Qlik forces you to retrieve data in chunks.</p>
<p>Understanding how to manage these chunks is the difference between an app that lags and one that delivers a good user experience.</p>
<h5><font color="#800080"><strong>Step 1: Defining the Data Volume</strong></font></h5>
<p>To see these strategies in action, we need a "heavy" dataset. Copy this script into your Qlik Sense Data Load Editor to generate 250,000 rows of transactions (or download the QVF attached to this post):</p>
// ============================================================
// DATASET GENERATOR: 250,000 rows (~1,000,000 cells)
// ============================================================

Transactions:
Load
    RecNo() as TransactionID,
    'Customer ' &amp; Ceil(Rand() * 20000) as Customer,
    Pick(Ceil(Rand() * 5), 
        'Corporate', 
        'Consumer', 
        'Small Business', 
        'Home Office', 
        'Enterprise'
    ) as Segment,
    Money(Rand() * 1000, '$#,##0.00') as Sales,
    Date(Today() - Rand() * 365) as [Transaction Date]
AutoGenerate 250000;
<h5><strong><font color="#800080">Step 2: Choosing Your Strategy</font></strong></h5>
<p>There are two primary ways to handle this volume in a web app. The choice depends entirely on your specific use case.</p>
<h5><font color="#339966"><strong>1- Bulk Ingest (The High-Performance Pattern)</strong></font></h5>
<p>In this pattern, you fetch the entire dataset into the application's local memory in iterative chunks upon loading.</p>
<ul>
<li>
<p><strong>The Goal:</strong> Provide a "zero-latency" experience once the data is loaded.</p>
</li>
<li>
<p><strong>Best For:</strong> Use cases where users need to perform instant client-side searches, complex local sorting, or full-dataset CSV exports without waiting for the Engine.</p>
</li>
</ul>
<p><span class="lia-inline-image-display-wrapper lia-image-align-inline" style="width: 999px;"><img alt="Ouadie_3-1767374031102.png" src="https://community.qlik.com/t5/image/serverpage/image-id/186035i7E80F4B0B645A84D/image-size/large?v=v2&amp;px=999" title="Ouadie_3-1767374031102.png" /></span></p>
<p>&nbsp;</p>
<h5><font color="#339966"><strong>2- On-Demand (The "Virtual" Pattern)</strong></font></h5>
<p>In this pattern, you only fetch the specific slice of data the user is currently looking at.</p>
<ul>
<li>
<p><strong>The Goal:</strong> Provide a near-instant initial load time, regardless of whether the dataset has 10,000 or 10,000,000 rows as you only load a specific chunk of those rows at a time.</p>
</li>
<li>
<p><strong>Best For:</strong> Massive datasets where the "cost" of loading everything into memory is too high, or when users only need to browse a few pages at a time.</p>
</li>
</ul>
<p><span class="lia-inline-image-display-wrapper lia-image-align-inline" style="width: 999px;"><img alt="Ouadie_2-1767373985587.png" src="https://community.qlik.com/t5/image/serverpage/image-id/186034iC298CF00B96D4F53/image-size/large?v=v2&amp;px=999" title="Ouadie_2-1767373985587.png" /></span></p>
<h5><strong><font color="#800080">Step 3: Implementing the Logic</font></strong></h5>
<p>While I'm using React and custom react hooks for the example I'm providing, these core Qlik concepts translate to any JavaScript framework (Vue, Angular, or Vanilla JS). The secret lies in how you interact with the HyperCube.</p>
<p><strong>The Iterative Logic (Bulk Ingest):</strong></p>
<p>The key is to use a loop that updates your local data buffer as chunks arrive.</p>
<p>To prevent the browser from freezing during this heavy network activity, we use <em>setTimeout</em> to allow the UI to paint the progress bar.</p>
        qModel = await app.createSessionObject({ qInfo: { qType: 'bulk' }, ...properties });
        const layout = await qModel.getLayout();
        const totalRows = layout.qHyperCube.qSize.qcy;
        const pageSize = properties.qHyperCubeDef.qInitialDataFetch[0].qHeight;
        const width = properties.qHyperCubeDef.qInitialDataFetch[0].qWidth;
        const totalPages = Math.ceil(totalRows / pageSize);

        let accumulator = [];
        for (let i = 0; i &lt; totalPages; i++) {
          if (!mountedRef.current || stopRequestedRef.current) break;

          const pages = await qModel.getHyperCubeData('/qHyperCubeDef', [{
            qTop: i * pageSize,
            qLeft: 0,
            qWidth: width,
            qHeight: pageSize
          }]);

          accumulator = accumulator.concat(pages[0].qMatrix);

          // Update state incrementally
          setData([...accumulator]);
          setProgress(Math.round(((i + 1) / totalPages) * 100));

          // Yield thread to prevent UI locking
          await new Promise(r =&gt; setTimeout(r, 1));
<p>&nbsp;</p>
<p><strong>The Slicing Logic (On-Demand)</strong></p>
<p>In this mode, the application logic simply calculates the <em>qTop</em> coordinate based on the user's current page index and makes a single request for that specific window of data (<em>rowsPerPage</em>).</p>
        const width = properties.qHyperCubeDef.qInitialDataFetch[0].qWidth;
        const qTop = (page - 1) * rowsPerPage;

        const pages = await qModelRef.current.getHyperCubeData('/qHyperCubeDef', [{
          qTop,
          qLeft: 0,
          qWidth: width,
          qHeight: rowsPerPage
        }]);

        if (mountedRef.current) {
          setData(pages[0].qMatrix);
        }
<p>&nbsp;</p>
<p>I placed these two methods in custom hooks (<strong>useQlikBulkIngest</strong> &amp; <strong>useQlikOnDemand</strong>) so they can be easily re-used in different components as well as other apps.</p>
<h5><font color="#800080"><strong>Best Practices</strong></font></h5>
<p>Regardless of which pattern you choose, always follow these three Qlik Engine best practices:</p>
<ol>
<li>
<p><strong>Engine Hygiene (Cleanup):</strong> Always call app.destroySessionObject(qModel.id) when your component or view unmounts.</p>
</li>
<li>
<p><strong>Cell Math:</strong> Always make sure your <strong>qWidth x qHeight </strong>is strictly<strong> &lt; 10,000</strong>. For instance, if you have a wide table (20 columns), your max height is only 500 rows per chunk.</p>
</li>
<li>
<p><strong>UI Performance:</strong> Even if you use the "Bulk" method and have 250,000 rows in JavaScript memory, <strong>do not render them all to the DOM at once</strong>. Use UI-level pagination or virtual scrolling to keep the browser responsive.</p>
</li>
</ol>
<p>Choosing between Bulk and On-Demand is a trade-off between <strong>Initial Load Time</strong> and <strong>Interactive Speed</strong>. By mastering iterative fetching with the <a href="https://community.qlik.com/t5/user/viewprofilepage/user-id/229862">@qlik</a>/api library, you can ensure your web apps remain robust, no matter how much data is coming in from Qlik.</p>
<p>Attached is the <strong>QVF</strong> and here is the&nbsp;<a href="https://github.com/olim-dev/qlik-pagination-demo" target="_self"><strong>GitHub repository containing the full example</strong></a> in React so you can try it in locally - <u>Instructions are provided in the <a href="https://github.com/olim-dev/qlik-pagination-demo/blob/main/README.md" target="_self">README file</a></u>.</p>
<p>(P.S:&nbsp;<em>Make sure you create the OAuth client in your tenant and fill in the qlik-config.js file in the project with your tenant-specific config).</em></p>
<p><strong>Thank you for reading!</strong></p>
