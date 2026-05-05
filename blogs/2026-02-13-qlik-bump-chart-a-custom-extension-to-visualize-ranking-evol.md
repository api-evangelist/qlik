---
title: "Qlik Bump Chart: a custom extension to visualize ranking evolution over time"
url: "https://community.qlik.com/t5/Design/Qlik-Bump-Chart-a-custom-extension-to-visualize-ranking/ba-p/2543027"
date: "Fri, 13 Feb 2026 23:22:55 GMT"
author: "Ouadie"
feed_url: "https://community.qlik.com:443/cyjdu72974/rss/board?board.id=qlik-design-blog"
---
<p><font face="arial,helvetica,sans-serif" size="3">When building dashboards that involve any kind of leaderboard or competitive comparison, it’s always good to show how rankings shift over time. Not just "who's #1 right now," but the full story: who climbed, who dropped, when the shift happened.&nbsp;</font></p>
<p><font face="arial,helvetica,sans-serif" size="3">Qlik Sense doesn't have a native bump chart. The usual workaround is a line chart with pre-calculated ranks, but I wanted to push this further and add some more interactivity and options.</font></p>
<p><font face="arial,helvetica,sans-serif" size="3">The idea for creating this extension came after doing some updates on the<a href="https://webapps.qlik.com/formula1/index.html#/races" target="_self"> Formula1 web app</a> which uses a native D3.js chart, so I wanted to transfer that code over to a Qlik Sense extension that can be reused in other apps.</font></p>
<p><font face="arial,helvetica,sans-serif" size="3">The extension makes it easy for you. You just add two dimensions and one measure, and it takes care of everything else: ranking, time sorting, labels, hover highlighting, and field selections. The object properties will let you customize it to your needs.</font></p>
<p><font face="arial,helvetica,sans-serif" size="3">In this post I'll walk through what it does, how it's structured, and how you can use it for various use cases.<br /><br /><strong>Github Repo:</strong>&nbsp;<a href="https://github.com/olim-dev/qlik-bump-chart" rel="noopener" target="_blank">https://github.com/olim-dev/qlik-bump-chart</a>&nbsp;</font></p>
<p><font color="#008000" face="arial,helvetica,sans-serif" size="3"><strong>What is a Bump Chart?</strong></font></p>
<p><font face="arial,helvetica,sans-serif" size="3">A bump chart shows how entities change position relative to each other over time. Each entity gets a colored line. Rank #1 sits at the top. As rankings shift, lines cross, and you immediately see who's moving up, who's falling, and when an overtake happens.</font><br /><br /></p>
<p><span class="lia-inline-image-display-wrapper lia-image-align-inline" style="width: 999px;"><img alt="Screenshot 2026-02-13 180717.png" src="https://community.qlik.com/t5/image/serverpage/image-id/186902iD062341A894AE1EE/image-size/large?v=v2&amp;px=999" title="Screenshot 2026-02-13 180717.png" /></span></p>
<p>&nbsp;</p>
<p><font color="#008000" face="arial,helvetica,sans-serif" size="3"><strong>What can you do with this extension?</strong></font></p>
<ul>
<li><font face="arial,helvetica,sans-serif" size="3">Track rankings over any time dimension (quarters, months, weeks, laps)</font></li>
<li>Highlight top performers, bottom performers, or biggest movers automatically</li>
<li>Toggle between rank mode (auto-calculated) and position mode (raw measure as Y-axis)</li>
<li>Replay the line-drawing animation</li>
<li>Click labels or lines to make Qlik selections</li>
<li>Customize everything: line style, dot size, colors, labels, grid, tooltips</li>
</ul>
<p>&nbsp;</p>
<p><font color="#008000" face="arial,helvetica,sans-serif" size="3"><strong>Use Cases</strong></font></p>
<p><font face="arial,helvetica,sans-serif" size="3">The pattern is always the same: entities, time periods, and a measure.</font></p>
<ul>
<li><font face="arial,helvetica,sans-serif" size="3">Sales: rep leaderboard over months</font></li>
<li>Marketing: brand awareness tracking, campaign effectiveness over quarters</li>
<li>Finance: fund performance rankings</li>
<li>HR: employee performance trends, team productivity rankings</li>
<li>Operations: supplier quality rankings, efficiency metric evolution</li>
<li>Sports: league standings, player rankings across a season, F1 race position over laps</li>
</ul>
<p><font face="arial,helvetica,sans-serif" size="3"><strong><span class="lia-inline-image-display-wrapper lia-image-align-inline" style="width: 999px;"><img alt="Screenshot 2026-02-13 174759.png" src="https://community.qlik.com/t5/image/serverpage/image-id/186900i2197972376D8B030/image-size/large?v=v2&amp;px=999" title="Screenshot 2026-02-13 174759.png" /></span></strong></font></p>
<p><font color="#008000" face="arial,helvetica,sans-serif" size="3"><strong>How to set it up</strong></font></p>
<p><font face="arial,helvetica,sans-serif" size="3">Add two dimensions and one measure:</font></p>
<ul>
<li><font face="arial,helvetica,sans-serif" size="3"><em><strong>Dimension 1:</strong></em> the entity to rank (Product, Sales Rep, Team, Driver)</font></li>
<li><em style="font-family: arial, helvetica, sans-serif; font-size: medium;"><strong>Dimension 2:</strong> </em><span>the time period (Quarter, Month, Week, Lap)</span></li>
<li><em style="font-family: arial, helvetica, sans-serif; font-size: medium;"><strong>Measure:</strong> </em><span>the value that determines rank (Sum(Sales), Avg(Score), Max(Position))</span></li>
</ul>
<p><font face="arial,helvetica,sans-serif" size="3"><br />I have attached a Text file with some sample datasets that you can inline-load in a new app to test it.</font></p>
<p>&nbsp;</p>
<p><font color="#008000" face="arial,helvetica,sans-serif" size="3"><strong>Quick tour of the object properties</strong></font></p>
<p><font face="arial,helvetica,sans-serif" size="3">The property panel has clear labels for which dimension goes on which axis. Here are the sections that matter most:</font></p>
<p><font face="arial,helvetica,sans-serif" size="3"><em><strong>Chart Settings:</strong></em> Maximum Entities (2-30, sweet spot is 8-15), Rank Direction (highest or lowest value = #1), Line Style (smooth, straight, step).</font></p>
<p><font face="arial,helvetica,sans-serif" size="3"><em><strong>Position Mode:</strong> </em>Instead of auto-calculating ranks, use the raw measure value as the Y position. Great for race data where position is already known.&nbsp;</font></p>
<p><font face="arial,helvetica,sans-serif" size="3"><em><strong>Labels:</strong></em> Position them on the right, left, or both sides. Rank change indicators (triangle arrows) show how many spots an entity moved. Clicking a label triggers a Qlik selection.</font></p>
<p><font face="arial,helvetica,sans-serif" size="3"><em><strong>Highlight:</strong></em> Automatically highlight top N, bottom N, or biggest movers with a configurable glow color.</font></p>
<p><font face="arial,helvetica,sans-serif" size="3"><em><strong>Colors:</strong> </em>Six built-in palettes (Vibrant, Qlik Classic, Category 10, Extended 20, Cool, Warm) plus a custom background color.</font></p>
<p><font face="arial,helvetica,sans-serif" size="3"><em><strong>Animation:</strong></em> Toggle transitions on/off, adjust duration (100-1500ms), and a "Replay Animation" button to re-run the line-drawing effect.</font></p>
<p><font face="arial,helvetica,sans-serif" size="3"><em><strong>Selections:</strong> </em>Toggle selections on/off</font></p>
<p>&nbsp;</p>
<p><font color="#008000" face="arial,helvetica,sans-serif" size="3"><strong>The extension folder structure</strong></font></p>
<p><font face="arial,helvetica,sans-serif" size="3">&nbsp; qlik-bump-chart/</font><br /><font face="arial,helvetica,sans-serif" size="3">&nbsp; &nbsp; &nbsp;qlik-bump-chart.qext&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;Extension definitions</font><br /><font face="arial,helvetica,sans-serif" size="3">&nbsp; &nbsp; &nbsp;qlik-bump-chart.js&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;Main entry (Qlik lifecycle, paint, selections)</font><br /><font face="arial,helvetica,sans-serif" size="3">&nbsp; &nbsp; &nbsp;qlik-bump-chart-properties.js&nbsp; &nbsp; &nbsp; &nbsp; Property panel definition</font><br /><font face="arial,helvetica,sans-serif" size="3">&nbsp; &nbsp; &nbsp;qlik-bump-chart-renderer.js&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;D3 rendering engine</font><br /><font face="arial,helvetica,sans-serif" size="3">&nbsp; &nbsp; &nbsp;qlik-bump-chart-data.js&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;Data processing, ranking, time sorting</font><br /><font face="arial,helvetica,sans-serif" size="3">&nbsp; &nbsp; &nbsp;qlik-bump-chart-constants.js&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;Defaults, palettes, timing values</font><br /><font face="arial,helvetica,sans-serif" size="3">&nbsp; &nbsp; &nbsp;qlik-bump-chart-style.css&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;All CSS</font></p>
<p><font face="arial,helvetica,sans-serif" size="3">lib/</font><br /><font face="arial,helvetica,sans-serif" size="3">&nbsp; &nbsp; &nbsp;d3.v7.min.js&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; D3.js v7</font></p>
<p>&nbsp;</p>
<ul>
<li><font face="arial,helvetica,sans-serif" size="3">The main JS handles the Qlik lifecycle.</font></li>
<li>The data module processes the hypercube, sorts time periods, and calculates ranks.</li>
<li>The renderer draws everything with D3.</li>
<li>The properties file defines the right-panel UI.</li>
<li>The constants file holds all defaults, color palettes, and timing values in one place.</li>
</ul>
<p>&nbsp;</p>
<p><font face="arial,helvetica,sans-serif" size="3"><strong>Tips</strong></font></p>
<ul>
<li><font face="arial,helvetica,sans-serif" size="3">Limit entities to 8-15 for readability. More than that and lines become hard to follow.</font></li>
<li>Use Highlight Mode to draw attention to the story (top performers, biggest movers).</li>
<li>Position Mode is great for pre-calculated position data like F1 race results.</li>
<li>Pair with a table or KPI object nearby for exact values at glance.</li>
</ul>
<p>&nbsp;</p>
<p><font face="arial,helvetica,sans-serif" size="3">Let me know in the comments if you have any questions or feedback!</font></p>
<p><font face="arial,helvetica,sans-serif" size="3">Thanks for reading.</font></p>
<p><font face="arial,helvetica,sans-serif" size="3">Ouadie</font></p>
