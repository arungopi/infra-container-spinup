Openwebui with Podman
1. For Ubuntu
   - ```bash 
        docker run -d -p 3000:8080 --add-host=host.docker.internal:host-gateway -e OLLAMA_BASE_URL=http://host.containers.internal:11434 -v open-webui:/app/backend/data --name open-webui --restart always ghcr.io/open-webui/open-webui:main
        ```
   - Refer : https://github.com/open-webui/open-webui
2. If Ollama is to accessible for Openwebui then it may not be avaiable on 0.0.0.0 then bind Ollama to http://0.0.0.0 : https://docs.ollama.com/faq#setting-environment-variables-on-linux
3. If above step is not working then :
   + add this to environment variable in /etc/systemd/system/ollama.service : "OLLAMA_HOST=0.0.0.0:11434" 
   + sudo systemctl daemon-reload
   + sudo systemctl restart ollama
4. For security add the below to explicilty allow local loop back traffic on 11434
   + sudo ufw allow in on lo to any port 11434   
     