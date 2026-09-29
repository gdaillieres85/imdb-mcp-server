import os
import uvicorn
from starlette.applications import Starlette
from starlette.routing import Route
from mcp.server.sse import SseServerTransport

# Importez l'instance de votre serveur MCP depuis votre code existant
# Adaptez 'main' au nom du fichier où votre variable 'server' (instance de mcp.Server) est créée.
from imdb_mcp_server.main import server 

# Initialise le transport SSE de MCP
sse = SseServerTransport("/messages")

async def handle_sse(request):
    """Gère la connexion initiale et maintient le flux SSE ouvert."""
    async with sse.connect_sse(
        request.scope, 
        request.receive, 
        request._send
    ) as streams:
        # Relie les flux web au serveur MCP
        await server.run(
            streams[0], 
            streams[1], 
            server.create_initialization_options()
        )

async def handle_messages(request):
    """Reçoit les requêtes (appels d'outils) envoyées par Salesforce/Claude."""
    await sse.handle_post_message(request.scope, request.receive, request._send)

# Crée l'application web avec les deux routes requises
app = Starlette(
    routes=[
        Route("/sse", endpoint=handle_sse),
        Route("/messages", endpoint=handle_messages, methods=["POST"]),
    ]
)

if __name__ == "__main__":
    # Render fournit dynamiquement le port via la variable d'environnement PORT
    port = int(os.environ.get("PORT", 8000))
    uvicorn.run(app, host="0.0.0.0", port=port)
