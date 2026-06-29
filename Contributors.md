---
layout: default
title: Contributors
nav_order: 3
description: "The team behind the Electromechanical Systems LAB Archive."
---

# Contributors
A special thank you to the contributors who maintain this archive.

<style>
  .team-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
    gap: 25px;
    padding: 20px 0;
  }
  .team-card {
    background: #ffffff;
    border: 1px solid #e1e4e8;
    border-radius: 12px;
    padding: 25px 15px;
    text-align: center;
    text-decoration: none !important;
    color: inherit;
    display: flex;
    flex-direction: column;
    align-items: center;
    transition: all 0.3s ease;
    box-shadow: 0 2px 5px rgba(0,0,0,0.05);
  }
  .team-card:hover {
    transform: translateY(-5px);
    box-shadow: 0 10px 20px rgba(0,0,0,0.15);
    border-color: #0366d6;
  }
  .team-avatar {
    width: 100px;
    height: 100px;
    border-radius: 50%;
    margin-bottom: 15px;
    object-fit: cover;
    border: 4px solid #f6f8fa;
  }
  .team-realname {
    font-size: 1.1em;
    font-weight: 700;
    color: #24292e;
    margin-bottom: 5px;
  }
  .team-username {
    font-size: 0.9em;
    color: #586069;
    font-weight: 400;
  }
</style>

<div id="contributors" class="team-grid">
  <div style="grid-column: 1 / -1; text-align: center; color: #666;">
    Loading team data...
  </div>
</div>

<script>
  document.addEventListener("DOMContentLoaded", function() {
    const repoOwner = 'em5l';
    const repoName = 'em5l.github.io';
    const container = document.getElementById('contributors');

    if (!container) {
      console.error("Error: The 'contributors' div is missing.");
      return;
    }

    fetch(`https://api.github.com/repos/${repoOwner}/${repoName}/contributors`)
      .then(response => {
        if (!response.ok) {
          if (response.status === 403) throw new Error("API Rate Limit Exceeded (Wait 1 hour)");
          if (response.status === 404) throw new Error("Repository not found (Check privacy settings)");
          throw new Error(`Error ${response.status}`);
        }
        return response.json();
      })
      .then(data => {
        container.innerHTML = ''; // Clear loading message

        if (data.length === 0) {
          container.innerHTML = '<p>No contributors found.</p>';
          return;
        }

        data.forEach(user => {
          if (user.type === 'Bot' || user.login.includes('[bot]')) return;

          const cardId = `user-${user.login}`; 
          
          const card = `
            <a href="${user.html_url}" class="team-card" id="${cardId}" target="_blank">
              <img class="team-avatar" src="${user.avatar_url}" alt="${user.login}">
              <div class="team-realname">${user.login}</div>
              <div class="team-username">@${user.login}</div>
            </a>
          `;
          container.innerHTML += card;

          // Fetch Real Name
          fetch(user.url)
            .then(resp => {
              if (!resp.ok) return null;
              return resp.json();
            })
            .then(profile => {
              if (profile && profile.name) {
                const nameDiv = document.getElementById(cardId).querySelector('.team-realname');
                if (nameDiv) nameDiv.innerText = profile.name;
              }
            })
            .catch(e => { /* Ignore errors */ });
        });
      })
      .catch(err => {
        container.innerHTML = `<p style="color:red; text-align:center;">${err.message}</p>`;
      });
  });
</script>