---
title: "Building a World Cup Bracket App powered by Qlik"
url: "https://community.qlik.com/t5/Design/Building-a-World-Cup-Bracket-App-powered-by-Qlik/ba-p/2547681"
date: "Fri, 24 Apr 2026 20:16:59 GMT"
author: "Ouadie"
feed_url: "https://community.qlik.com:443/cyjdu72974/rss/board?board.id=qlik-design-blog"
---
<p class="p1">With less than 50 days to go before the 2026 World Cup kicks off across the US, Canada, and Mexico, I wanted to share a project I've been working on that brings together a few pieces of the Qlik platform I think work really well together: Choose Your Champion 2026.</p>
<p class="p1">It's a web app where anyone can fill out their World Cup bracket, get AI-powered predictions for every possible matchup in the tournament powered by Qlik Predict, explore historical World Cup data, and compete on a leaderboard as the competition unfolds.</p>
<p class="p2"><span class="s1">You can try it here:&nbsp;<a href="https://webapps.qlik.com/choose-your-champion-2026/index.html#/" target="_blank">https://webapps.qlik.com/choose-your-champion-2026/index.html#/</a>&nbsp;</span></p>
<p class="p1"><span class="lia-inline-image-display-wrapper lia-image-align-inline" style="width: 548px;"><img alt="Screenshot 2026-04-24 at 7.11.24 AM.png" height="304" src="https://community.qlik.com/t5/image/serverpage/image-id/187985i5F1AF02DEF55F545/image-dimensions/548x304?v=v2" title="Screenshot 2026-04-24 at 7.11.24 AM.png" width="548" /></span></p>
<p class="p1">The app is powered by Qlik, with Qlik Cloud Analytics for the data model and Historical Analysis, Qlik Predict for the matchup predictions, and various Qlik APIs to wire everything into a React front-end.</p>
<p class="p1">In this post, I'll walk through how the predictions work under the hood, because that was the most interesting piece to build.</p>
<h4 class="p3"><font color="#339966"><strong>What's in the app:</strong></font></h4>
<p class="p3">Choose Your Champion is broken into 4 parts:</p>
<ul class="ul1">
<li class="li1"><strong><i>Build a bracket:</i></strong> Pick your group stage winners, advance teams through the knockout rounds, and lock in your champion.</li>
</ul>
<p>&nbsp;</p>
<ul class="ul1">
<li class="li1"><strong><i>Check the predictions:</i></strong> For every possible matchup in the tournament, the app surfaces a Qlik Predict generated win probability for each team plus a draw probability. When you're unsure about a matchup, you can pull up the prediction and use it to decide which team advances.</li>
</ul>
<p class="p1"><span class="lia-inline-image-display-wrapper lia-image-align-inline" style="width: 444px;"><img alt="Screenshot 2026-04-24 at 7.13.20 AM.png" height="213" src="https://community.qlik.com/t5/image/serverpage/image-id/187987i4AD2F5B5646C15BD/image-dimensions/444x213?v=v2" title="Screenshot 2026-04-24 at 7.13.20 AM.png" width="444" /></span></p>
<p class="p1"><span class="lia-inline-image-display-wrapper lia-image-align-inline" style="width: 334px;"><img alt="Screenshot 2026-04-24 at 7.13.38 AM.png" height="313" src="https://community.qlik.com/t5/image/serverpage/image-id/187986i28DC34803560AFFB/image-dimensions/334x313?v=v2" title="Screenshot 2026-04-24 at 7.13.38 AM.png" width="334" /></span></p>
<p class="p1">&nbsp;</p>
<ul class="ul1">
<li class="li1"><strong><i>Explore historical World Cup data: </i></strong>The app includes various visualizations to help you uncover insights from past tournaments: goals, top scorers, host nation performance, biggest upsets. All powered by the associative engine.</li>
</ul>
<p><span class="lia-inline-image-display-wrapper lia-image-align-inline" style="width: 400px;"><img alt="Screenshot 2026-04-24 at 7.16.20 AM.png" src="https://community.qlik.com/t5/image/serverpage/image-id/187988i2718524938CC435E/image-size/medium?v=v2&amp;px=400" title="Screenshot 2026-04-24 at 7.16.20 AM.png" /></span></p>
<p>&nbsp;</p>
<ul class="ul1">
<li class="li1"><strong><i>Leaderboard: </i></strong>As real matches get played in June and July, submitted brackets are scored automatically and players are ranked in the leaderboard table.</li>
</ul>
<p><span class="lia-inline-image-display-wrapper lia-image-align-inline" style="width: 400px;"><img alt="Screenshot 2026-04-24 at 7.18.31 AM.png" src="https://community.qlik.com/t5/image/serverpage/image-id/187991i721F273AF03B79E8/image-size/medium?v=v2&amp;px=400" title="Screenshot 2026-04-24 at 7.18.31 AM.png" /></span></p>
<p>&nbsp;</p>
<h4 class="p3"><font color="#339966"><strong>Under the hood: how the predictions work</strong></font></h4>
<p class="p1">This was the fun part. The goal was simple, given two national teams, predict the outcome of a hypothetical match (team A wins / draw / team B wins), but the work that makes the predictions actually useful is mostly in the data, not the model (thanks to no-code ML with Qlik Predict).</p>
<p class="p5"><font color="#800080"><strong><i>1. The training dataset</i></strong></font></p>
<p class="p1">I started with every international football match result from 1872 to March 2026. There's a well-maintained open dataset on GitHub (credit:&nbsp;<a href="https://github.com/martj42/international_results" rel="noopener" target="_blank"><span class="s3">martj42/international_results</span></a>) that gets updated after every international window, about 49,000 matches in total.</p>
<p class="p1">From that raw history, I built a training dataset focused on the modern era (2010 onwards) and only competitive matches (qualifiers, continental tournaments, World Cup finals). Friendlies got filtered out because they're noisy since teams often don't play their A squads, and the stakes don't match what happens in a real tournament.</p>
<p class="p1">That left me with around 9,400 training rows, each representing a real historical match with a known result, enriched with 27 features describing both teams' state going into that match:</p>
<ul class="ul1">
<li class="li6">Elo ratings for both teams</li>
<li class="li6">FIFA rankings and points snapshot to the match date</li>
<li class="li6">Rolling 10-match form per team: win rate, goals for, goals against, goal difference</li>
<li class="li6">Head-to-head history in the last 10 meetings</li>
<li class="li6">Context flags: neutral venue, tournament tier, cross-confederation</li>
<li class="li6">World Cup pedigree: a score rewarding teams for deep runs in past tournaments, with more recent success weighted heavier</li>
</ul>
<p class="p7">&nbsp;</p>
<p class="p7"><font color="#800080"><strong><i>2. ML Experiment</i></strong></font></p>
<p class="p1">Once the training CSV was in shape, I uploaded it to Qlik Predict, pointed at the result column as the target, and let it do its thing. This is where Qlik Predict really shines, zero code needed. No Python notebooks, no sklearn, no hyperparameter grids to tune. You just upload your data, pick a target, and it does the heavy lifting with full explainability on the outcomes and what drives the predictions.</p>
<p class="p1">Qlik Predict runs multiple algorithms in parallel: LightGBM, CatBoost, XGBoost, Random Forest, and a few others, tunes their hyperparameters, and picks the best performer by F1.</p>
<p class="p1"><span class="lia-inline-image-display-wrapper lia-image-align-inline" style="width: 515px;"><img alt="Screenshot 2026-04-24 at 12.15.23 AM.png" height="296" src="https://community.qlik.com/t5/image/serverpage/image-id/187992i72CBB55223AFB9E5/image-dimensions/515x296?v=v2" title="Screenshot 2026-04-24 at 12.15.23 AM.png" width="515" /></span></p>
<p class="p1">On my first run, I left all the columns in the dataset checked, including the team name columns (team_a, team_b). When I looked at the SHAP importance chart afterward, team_b and team_a were ranking as the #2 and #3 most influential features, meaning the model was essentially learning "team X usually wins" rather than learning from the engineered features.</p>
<p class="p1">I created a new version, went back to the Data tab, unchecked the team name columns and a few date fields (which were also ranking higher than they should), and re-ran the experiment. Qlik Predict automatically dropped several more low-importance features during training, leaving a clean, focused feature set. The F1 did not change a lot (stayed at ~0.50), but the SHAP chart now showed the model leaning on exactly the signals we want:</p>
<ol class="ol1">
<li class="li6">elo_diff</li>
<li class="li6">rank_diff</li>
<li class="li6">is_neutral</li>
<li class="li6">h2h_team_a_advantage<br />etc...</li>
</ol>
<p class="p1"><span class="lia-inline-image-display-wrapper lia-image-align-inline" style="width: 999px;"><img alt="Screenshot 2026-04-24 at 12.43.16 AM.png" src="https://community.qlik.com/t5/image/serverpage/image-id/187993iED258577A3D358CA/image-size/large?v=v2&amp;px=999" title="Screenshot 2026-04-24 at 12.43.16 AM.png" /></span></p>
<p class="p1">&nbsp;</p>
<p class="p1">A few other calls that mattered:</p>
<ul class="ul1">
<li class="li6"><strong>Filtering to competitive matches only.</strong> A friendly between a top side's B squad and a mid-tier opponent tells you almost nothing about what happens in a World Cup group stage game.</li>
<li class="li6"><strong>Exponential decay on World Cup pedigree.</strong> A deep run in 1970 still counts, but less than one in 2022.</li>
<li class="li6"><strong>Removing rows with too many missing features.</strong> FIFA rankings don't go back to the 90s for every team, so some rows had to get dropped.</li>
</ul>
<p class="p4">&nbsp;</p>
<p class="p5"><font color="#800080"><strong><i>3. The apply dataset</i></strong></font></p>
<p class="p1">Training gives you a model and to use it, you need an apply dataset with new rows you want predictions for.</p>
<p class="p1">For Choose Your Champion, I generated every possible pairing of the 48 qualified teams, which comes out to 1,128 unique matchups. Each row has the same 27 features as the training dataset, but computed as a current snapshot: each team's Elo today, their current FIFA ranking, their most recent 10-match form, and so on.</p>
<p class="p1">I fed that into the deployed model and got back a probability distribution for every matchup: P(team_a_win), P(draw), P(team_b_win).</p>
<p class="p1"><span class="lia-inline-image-display-wrapper lia-image-align-inline" style="width: 561px;"><img alt="Screenshot 2026-04-24 at 7.35.44 AM.png" height="262" src="https://community.qlik.com/t5/image/serverpage/image-id/187994iB0E75531E56E82AA/image-dimensions/561x262?v=v2" title="Screenshot 2026-04-24 at 7.35.44 AM.png" width="561" /></span></p>
<h4 class="p5"><font color="#339966"><strong>The web app</strong></font></h4>
<p class="p1">The web app is a React front-end that connects to the Qlik tenant over anonymous access via <a href="https://qlik.dev/apis/" target="_self">@qlik/</a><a href="https://qlik.dev/apis/" target="_self">api</a>, so users never see a login screen or have to authenticate against a tenant. The bracket UI pulls predictions from the Qlik Sense data model, so whenever a user opens a matchup, they're looking at data straight from Qlik.</p>
<p class="p1">For the historical World Cup section, I used a mix of <a href="https://community.qlik.com/t5/Design/Discovering-qlik-embed-Qlik-s-new-library-for-Embedding-Qlik/ba-p/2141202" target="_self">@qlik/embed</a> components when I needed a quick, ready-to-use chart, and custom <a href="https://community.qlik.com/t5/Design/Using-Nebula-js-amp-D3-js-to-build-a-visualization-extension-for/ba-p/2050662" target="_self">nebula.js + picasso.js</a> visualizations when I needed more control over the styling to match the app's look and feel. Both approaches work against the same underlying Qlik Analytics app, so everything stays consistent and governed in one place.</p>
<p class="p1"><span class="lia-inline-image-display-wrapper lia-image-align-inline" style="width: 999px;"><img alt="Screenshot 2026-04-24 at 7.53.56 AM.png" src="https://community.qlik.com/t5/image/serverpage/image-id/187995i11DAAAD89EE797C1/image-size/large?v=v2&amp;px=999" title="Screenshot 2026-04-24 at 7.53.56 AM.png" /></span></p>
<h4 class="p3">&nbsp;</h4>
<h4 class="p3"><font color="#339966"><strong>A few takeaways</strong></font></h4>
<p class="p1">If you're thinking about building something similar, a few things worth keeping in mind:</p>
<p class="p1">Spend the time on feature engineering. The difference between a model that predicts noise and one that predicts football is almost entirely in the features. Qlik Predict handles algorithm selection and tuning well, but it can only work with what you feed it.</p>
<p class="p1">The integration is where Qlik Predict pays off. Once a model is deployed, scoring a new dataset and pulling scores back into a Qlik Cloud Analytics app takes one load script. No Python services to maintain, no separate MLOps platform to stand up, no JSON plumbing between systems. That end-to-end data prep, modeling, predictions, and analytics all living in one platform is the thing that made this project come together fast!</p>
<p class="p3"><font color="#339966"><a href="https://webapps.qlik.com/choose-your-champion-2026/index.html" rel="noopener" target="_blank"><strong>Go fill out your bracket</strong></a></font></p>
<p class="p1">The World Cup starts June 11, so there's plenty of time to get your bracket in and earn your spot on the leaderboard before kickoff. If you're curious about how any of this was built, leave a comment or reach out to me directly!</p>
<p class="p1">And if you want to learn more about Qlik Predict and start using it, visit:&nbsp;<a href="https://www.qlik.com/us/products/qlik-predict" rel="noopener" target="_blank">https://www.qlik.com/us/products/qlik-predict</a>&nbsp;</p>
<p class="p1"><strong>P.S:</strong> I have attached both <em>Training</em> and <em>Apply</em> datasets if you'd like to use them in your own Qlik Predict experiment.</p>
<p class="p1"><br />Thank you!</p>
