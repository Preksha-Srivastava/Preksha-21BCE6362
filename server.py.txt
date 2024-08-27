import asyncio
import websockets
import json

# Game state
class Game:
    def __init__(self):
        self.players = {}
        self.board = [['' for _ in range(5)] for _ in range(5)]
        self.current_turn = 'A'
        self.game_over = False

    def setup_game(self, player_id, characters):
        if player_id not in self.players:
            self.players[player_id] = characters
            row = 0 if player_id == 'A' else 4
            for i in range(5):
                self.board[row][i] = f"{player_id}-{characters[i]}"

    def get_valid_moves(self, player_id, character, row, col):
        valid_moves = []
        if 'P' in character:  # Pawn
            if player_id == 'B':
                if row > 0: valid_moves.append('F')  # Forward for A (up)
                if row < 4: valid_moves.append('B')  # Backward for A (down)
                if col > 0: valid_moves.append('L')  # Left
                if col < 4: valid_moves.append('R')  # Right
            else:  # Player A
                if row < 4: valid_moves.append('F')  # Forward for B (down)
                if row > 0: valid_moves.append('B')  # Backward for B (up)
                if col > 0: valid_moves.append('R')  # Left
                if col < 4: valid_moves.append('L')  # Right
        
        elif 'H1' in character:  # Hero1
            if player_id == 'B':
                if row > 1: valid_moves.append('F')  # Forward for A (up)
                if row < 3: valid_moves.append('B')  # Backward for A (down)
                if col > 1: valid_moves.append('L')
                if col < 3: valid_moves.append('R')
            else:  # Player A
                if row < 3: valid_moves.append('F')  # Forward for B (down)
                if row > 1: valid_moves.append('B')  # Backward for B (up)
                if col > 1: valid_moves.append('L')
                if col < 3: valid_moves.append('R')
        elif 'H2' in character:  # Hero2
            if row > 1 and col > 1: valid_moves.append('FL')
            if row > 1 and col < 3: valid_moves.append('FR')
            if row < 3 and col > 1: valid_moves.append('BL')
            if row < 3 and col < 3: valid_moves.append('BR')
        return valid_moves
    

    '''def get_valid_moves(self, player_id, character, row, col):
        valid_moves = []
        if 'P' in character:  # Pawn
            if col > 0: valid_moves.append('L')
            if col < 4: valid_moves.append('R')
            if row > 0: valid_moves.append('F')  # Forward
            if row < 4: valid_moves.append('B')  # Backward
        elif 'H1' in character:  # Hero1
            if col > 1: valid_moves.append('L')
            if col < 3: valid_moves.append('R')
            if row > 1: valid_moves.append('F')
            if row < 3: valid_moves.append('B')
        elif 'H2' in character:  # Hero2
            if row > 1 and col > 1: valid_moves.append('FL')
            if row > 1 and col < 3: valid_moves.append('FR')
            if row < 3 and col > 1: valid_moves.append('BL')
            if row < 3 and col < 3: valid_moves.append('BR')
        return valid_moves'''

    def validate_move(self, player_id, character, move):
        if self.current_turn != player_id:
            return False, "Not your turn."
        
        # Find the character on the board
        char_position = None
        for r in range(5):
            for c in range(5):
                if self.board[r][c] == f"{player_id}-{character}":
                    char_position = (r, c)
                    break
            if char_position:
                break
        if not char_position:
            return False, "Character not found."

        row, col = char_position
        # Handle movement based on character type
        if 'P' in character:  # Pawn
            if player_id == 'B':
                # A's moves
                if move == 'L' and col > 0:
                    col -= 1
                elif move == 'R' and col < 4:
                    col += 1
                elif move == 'F' and row > 0:  # Forward for A is up
                    row -= 1
                elif move == 'B' and row < 4:  # Backward for A is down
                    row += 1
                else:
                    return False, "Invalid move."
            else:  # Player A
                # B's moves
                if move == 'L' and col < 4:
                    col += 1
                elif move == 'R' and col > 0:
                    col -= 1
                elif move == 'F' and row < 4:  # Forward for B is down
                    row += 1
                elif move == 'B' and row > 0:  # Backward for B is up
                    row -= 1
                else:
                    return False, "Invalid move."
        elif 'H1' in character:  # Hero1
            if player_id == 'B':
                if move == 'L' and col > 1:
                    col -= 2
                elif move == 'R' and col < 3:
                    col += 2
                elif move == 'F' and row > 1:
                    row -= 2
                elif move == 'B' and row < 3:
                    row += 2
                else:
                    return False, "Invalid move."
            else:  # Player A
                if move == 'L' and col < 3:
                    col += 2
                elif move == 'R' and col > 1:
                    col -= 2
                elif move == 'F' and row < 3:
                    row += 2
                elif move == 'B' and row > 1:
                    row -= 2
                else:
                    return False, "Invalid move."
        elif 'H2' in character:  # Hero2
            if player_id == 'B':
                if move == 'FL' and row > 1 and col > 1:
                    row -= 2
                    col -= 2
                elif move == 'FR' and row > 1 and col < 3:
                    row -= 2
                    col += 2
                elif move == 'BL' and row < 3 and col > 1:
                    row += 2
                    col -= 2
                elif move == 'BR' and row < 3 and col < 3:
                    row += 2
                    col += 2
                else:
                    return False, "Invalid move."
            else:  # Player A
                if move == 'FL' and row < 3 and col < 3:
                    row += 2
                    col += 2
                elif move == 'FR' and row < 3 and col > 1:
                    row += 2
                    col -= 2
                elif move == 'BL' and row > 1 and col < 3:
                    row -= 2
                    col += 2
                elif move == 'BR' and row > 1 and col > 1:
                    row -= 2
                    col -= 2
                else:
                    return False, "Invalid move."
        else:
            return False, "Invalid character type."

        # Check for friendly fire
        if self.board[row][col].startswith(player_id):
            return False, "Cannot move onto a friendly piece."
        
        # Handle capturing
        if self.board[row][col] and not self.board[row][col].startswith(player_id):
            self.board[row][col] = ''  # Capture opponent's piece

        # Update the board
        old_row, old_col = char_position
        self.board[old_row][old_col] = ''
        self.board[row][col] = f"{player_id}-{character}"

        # Check for game over condition
        if all(self.board[r][c] == '' for r in range(5) for c in range(5) if self.board[r][c].startswith('B' if player_id == 'A' else 'A')):
            self.game_over = True
            return True, f"Player {player_id} wins!"

        # Switch turns
        self.current_turn = 'B' if self.current_turn == 'A' else 'A'
        return True, "Move successful."

game = Game()

async def handle_client(websocket, path):
    async for message in websocket:
        data = json.loads(message)

        if data['type'] == 'setup':
            player_id = data['player']
            characters = data['characters']
            game.setup_game(player_id, characters)
            await websocket.send(json.dumps({'type': 'state', 'board': game.board, 'turn': game.current_turn}))
        
        elif data['type'] == 'select':
            player_id = data['player']
            row = data['row']
            col = data['col']
            char = None
            for r in range(5):
                for c in range(5):
                    if game.board[r][c].startswith(player_id):
                        char = game.board[r][c].split('-')[1]
                        break
                if char:
                    break
            if char:
                valid_moves = game.get_valid_moves(player_id, char, row, col)
                await websocket.send(json.dumps({'type': 'valid_moves', 'moves': valid_moves}))

        elif data['type'] == 'move':
            player_id = data['player']
            character = data['character']
            move = data['move']
            valid, message = game.validate_move(player_id, character, move)
            if valid:
                if game.game_over:
                    await websocket.send(json.dumps({'type': 'game_over', 'message': message}))
                else:
                    await websocket.send(json.dumps({'type': 'state', 'board': game.board, 'turn': game.current_turn}))
            else:
                await websocket.send(json.dumps({'type': 'invalid_move', 'message': message}))

async def main():
    async with websockets.serve(handle_client, "localhost", 8765):
        await asyncio.Future()  # Run forever

asyncio.run(main())


'''import asyncio
import websockets
import json

# Game state
class Game:
    def __init__(self):
        self.players = {}
        self.board = [['' for _ in range(5)] for _ in range(5)]
        self.current_turn = 'A'
        self.game_over = False

    def setup_game(self, player_id, characters):
        if player_id not in self.players:
            self.players[player_id] = characters
            row = 0 if player_id == 'A' else 4
            for i in range(5):
                self.board[row][i] = f"{player_id}-{characters[i]}"

    def get_valid_moves(self, player_id, character, row, col):
        valid_moves = []
        if 'P' in character:  # Pawn
            if col > 0: valid_moves.append('L')
            if col < 4: valid_moves.append('R')
            if row > 0: valid_moves.append('F')
            if row < 4: valid_moves.append('B')
        elif 'H1' in character:  # Hero1
            if col > 1: valid_moves.append('L')
            if col < 3: valid_moves.append('R')
            if row > 1: valid_moves.append('F')
            if row < 3: valid_moves.append('B')
        elif 'H2' in character:  # Hero2
            if row > 1 and col > 1: valid_moves.append('FL')
            if row > 1 and col < 3: valid_moves.append('FR')
            if row < 3 and col > 1: valid_moves.append('BL')
            if row < 3 and col < 3: valid_moves.append('BR')
        return valid_moves

    def validate_move(self, player_id, character, move):
        if self.current_turn != player_id:
            return False, "Not your turn."
        
        # Find the character on the board
        char_position = None
        for r in range(5):
            for c in range(5):
                if self.board[r][c] == f"{player_id}-{character}":
                    char_position = (r, c)
                    break
            if char_position:
                break
        if not char_position:
            return False, "Character not found."

        row, col = char_position
        # Handle movement based on character type
        if 'P' in character:  # Pawn
            if move == 'L' and col > 0:
                col -= 1
            elif move == 'R' and col < 4:
                col += 1
            elif move == 'F' and row > 0:
                row -= 1
            elif move == 'B' and row < 4:
                row += 1
            else:
                return False, "Invalid move."
        elif 'H1' in character:  # Hero1
            if move == 'L' and col > 1:
                col -= 2
            elif move == 'R' and col < 3:
                col += 2
            elif move == 'F' and row > 1:
                row -= 2
            elif move == 'B' and row < 3:
                row += 2
            else:
                return False, "Invalid move."
        elif 'H2' in character:  # Hero2
            if move == 'FL' and row > 1 and col > 1:
                row -= 2
                col -= 2
            elif move == 'FR' and row > 1 and col < 3:
                row -= 2
                col += 2
            elif move == 'BL' and row < 3 and col > 1:
                row += 2
                col -= 2
            elif move == 'BR' and row < 3 and col < 3:
                row += 2
                col += 2
            else:
                return False, "Invalid move."
        else:
            return False, "Invalid character type."

        # Check for friendly fire
        if self.board[row][col].startswith(player_id):
            return False, "Cannot move onto a friendly piece."
        
        # Handle capturing
        if self.board[row][col] and not self.board[row][col].startswith(player_id):
            self.board[row][col] = ''  # Capture opponent's piece

        # Update the board
        old_row, old_col = char_position
        self.board[old_row][old_col] = ''
        self.board[row][col] = f"{player_id}-{character}"

        # Check for game over condition
        if all(self.board[r][c] == '' for r in range(5) for c in range(5) if self.board[r][c].startswith('B' if player_id == 'A' else 'A')):
            self.game_over = True
            return True, f"Player {player_id} wins!"

        # Switch turns
        self.current_turn = 'B' if self.current_turn == 'A' else 'A'
        return True, "Move successful."

game = Game()

async def handle_client(websocket, path):
    async for message in websocket:
        data = json.loads(message)

        if data['type'] == 'setup':
            player_id = data['player']
            characters = data['characters']
            game.setup_game(player_id, characters)
            await websocket.send(json.dumps({'type': 'state', 'board': game.board, 'turn': game.current_turn}))
        
        elif data['type'] == 'select':
            player_id = data['player']
            row = data['row']
            col = data['col']
            char = None
            for r in range(5):
                for c in range(5):
                    if game.board[r][c].startswith(player_id):
                        char = game.board[r][c].split('-')[1]
                        break
                if char:
                    break
            if char:
                valid_moves = game.get_valid_moves(player_id, char, row, col)
                await websocket.send(json.dumps({'type': 'valid_moves', 'moves': valid_moves}))

        elif data['type'] == 'move':
            player_id = data['player']
            character = data['character']
            move = data['move']
            valid, message = game.validate_move(player_id, character, move)
            if valid:
                if game.game_over:
                    await websocket.send(json.dumps({'type': 'game_over', 'message': message}))
                else:
                    await websocket.send(json.dumps({'type': 'state', 'board': game.board, 'turn': game.current_turn}))
            else:
                await websocket.send(json.dumps({'type': 'invalid_move', 'message': message}))

async def main():
    async with websockets.serve(handle_client, "localhost", 8765):
        await asyncio.Future()  # Run forever

asyncio.run(main())'''