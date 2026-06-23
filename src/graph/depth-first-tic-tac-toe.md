# Поиск в глубину — Крестики-нолики

```rust
#[derive(Copy, Clone, PartialEq, Debug)]
struct Position {
    x: u8,
    y: u8,
}

#[derive(Copy, Clone, PartialEq, Debug)]
pub enum Players {
    Blank,
    PlayerX,
    PlayerO,
}

#[derive(Copy, Clone, PartialEq, Debug)]
struct SinglePlayAction {
    position: Position,
    side: Players,
}

#[derive(Clone, PartialEq, Debug)]
pub struct PlayActions {
    positions: Vec<Position>,
    side: Players,
}

#[allow(dead_code)]
fn display_board(board: &[Vec<Players>]) {
    println!();
    for (y, board_row) in board.iter().enumerate() {
        print!("{} ", (y + 1));
        for board_cell in board_row {
            match board_cell {
                Players::PlayerX => print!("X "),
                Players::PlayerO => print!("O "),
                Players::Blank => print!("_ "),
            }
        }
        println!();
    }
    println!("  a b c");
}

fn available_positions(board: &[Vec<Players>]) -> Vec<Position> {
    let mut available: Vec<Position> = Vec::new();
    for (y, board_row) in board.iter().enumerate() {
        for (x, board_cell) in board_row.iter().enumerate() {
            if *board_cell == Players::Blank {
                available.push(Position {
                    x: x as u8,
                    y: y as u8,
                });
            }
        }
    }
    available
}

fn win_check(player: Players, board: &[Vec<Players>]) -> bool {
    if player == Players::Blank {
        return false;
    }

    //Check for a win on the diagonals.
    if (board[0][0] == board[1][1]) && (board[1][1] == board[2][2]) && (board[2][2] == player)
        || (board[2][0] == board[1][1]) && (board[1][1] == board[0][2]) && (board[0][2] == player)
    {
        return true;
    }

    for i in 0..3 {
        //Check for a win on the horizontals.
        if (board[i][0] == board[i][1]) && (board[i][1] == board[i][2]) && (board[i][2] == player) {
            return true;
        }

        //Check for a win on the verticals.
        if (board[0][i] == board[1][i]) && (board[1][i] == board[2][i]) && (board[2][i] == player) {
            return true;
        }
    }

    false
}

//Minimize the actions of the opponent while maximizing the game state of the current player.
pub fn minimax(side: Players, board: &[Vec<Players>]) -> Option<PlayActions> {
    //Check that board is in a valid state.
    if win_check(Players::PlayerX, board) || win_check(Players::PlayerO, board) {
        return None;
    }

    let opposite = match side {
        Players::PlayerX => Players::PlayerO,
        Players::PlayerO => Players::PlayerX,
        Players::Blank => panic!("Minimax can't operate when a player isn't specified."),
    };

    let positions = available_positions(board);
    if positions.is_empty() {
        return None;
    }

    //Play position
    let mut best_move: Option<PlayActions> = None;

    for pos in positions {
        let mut board_next = board.to_owned();
        board_next[pos.y as usize][pos.x as usize] = side;

        //Check for a win condition before recursion to determine if this node is terminal.
        if win_check(Players::PlayerX, &board_next) {
            append_playaction(
                side,
                &mut best_move,
                SinglePlayAction {
                    position: pos,
                    side: Players::PlayerX,
                },
            );
            continue;
        }

        if win_check(Players::PlayerO, &board_next) {
            append_playaction(
                side,
                &mut best_move,
                SinglePlayAction {
                    position: pos,
                    side: Players::PlayerO,
                },
            );
            continue;
        }

        let result = minimax(opposite, &board_next);
        let current_score = match result {
            Some(x) => x.side,
            _ => Players::Blank,
        };

        append_playaction(
            side,
            &mut best_move,
            SinglePlayAction {
                position: pos,
                side: current_score,
            },
        )
    }
    best_move
}

//Promote only better or collate equally scored game plays
fn append_playaction(
    current_side: Players,
    opt_play_actions: &mut Option<PlayActions>,
    appendee: SinglePlayAction,
) {
    if opt_play_actions.is_none() {
        *opt_play_actions = Some(PlayActions {
            positions: vec![appendee.position],
            side: appendee.side,
        });
        return;
    }

    let mut play_actions = opt_play_actions.as_mut().unwrap();

    //New game action is scored from the current side and the current saved best score against the new game action.
    match (current_side, play_actions.side, appendee.side) {
        (Players::Blank, _, _) => panic!("Unreachable state."),

        //Winning scores
        (Players::PlayerX, Players::PlayerX, Players::PlayerX) => {
            play_actions.positions.push(appendee.position);
        }
        (Players::PlayerX, Players::PlayerX, _) => {}
        (Players::PlayerO, Players::PlayerO, Players::PlayerO) => {
            play_actions.positions.push(appendee.position);
        }
        (Players::PlayerO, Players::PlayerO, _) => {}

        //Non-winning to Winning scores
        (Players::PlayerX, _, Players::PlayerX) => {
            play_actions.side = Players::PlayerX;
            play_actions.positions.clear();
            play_actions.positions.push(appendee.position);
        }
        (Players::PlayerO, _, Players::PlayerO) => {
            play_actions.side = Players::PlayerO;
            play_actions.positions.clear();
            play_actions.positions.push(appendee.position);
        }

        //Losing to Neutral scores
        (Players::PlayerX, Players::PlayerO, Players::Blank) => {
            play_actions.side = Players::Blank;
            play_actions.positions.clear();
            play_actions.positions.push(appendee.position);
        }

        (Players::PlayerO, Players::PlayerX, Players::Blank) => {
            play_actions.side = Players::Blank;
            play_actions.positions.clear();
            play_actions.positions.push(appendee.position);
        }

        //Ignoring lower scored plays
        (Players::PlayerX, Players::Blank, Players::PlayerO) => {}
        (Players::PlayerO, Players::Blank, Players::PlayerX) => {}

        //No change hence append only
        (_, _, _) => {
            assert!(play_actions.side == appendee.side);
            play_actions.positions.push(appendee.position);
        }
    }
}

#[cfg(test)]
mod test {
    use super::*;

    #[test]
    fn win_state_check() {
        let mut board = vec![vec![Players::Blank; 3]; 3];
        board[0][0] = Players::PlayerX;
        board[0][1] = Players::PlayerX;
        board[0][2] = Players::PlayerX;
        let responses = minimax(Players::PlayerO, &board);
        assert_eq!(responses, None);
    }

    #[test]
    fn win_state_check2() {
        let mut board = vec![vec![Players::Blank; 3]; 3];
        board[0][0] = Players::PlayerX;
        board[0][1] = Players::PlayerO;
        board[1][0] = Players::PlayerX;
        board[1][1] = Players::PlayerO;
        board[2][1] = Players::PlayerO;
        let responses = minimax(Players::PlayerO, &board);
        assert_eq!(responses, None);
    }

    #[test]
    fn block_win_move() {
        let mut board = vec![vec![Players::Blank; 3]; 3];
        board[0][0] = Players::PlayerX;
        board[0][1] = Players::PlayerX;
        board[1][2] = Players::PlayerO;
        board[2][2] = Players::PlayerO;
        let responses = minimax(Players::PlayerX, &board);
        assert_eq!(
            responses,
            Some(PlayActions {
                positions: vec![Position { x: 2, y: 0 }],
                side: Players::PlayerX
            })
        );
    }

    #[test]
    fn block_move() {
        let mut board = vec![vec![Players::Blank; 3]; 3];
        board[0][1] = Players::PlayerX;
        board[0][2] = Players::PlayerO;
        board[2][0] = Players::PlayerO;
        let responses = minimax(Players::PlayerX, &board);
        assert_eq!(
            responses,
            Some(PlayActions {
                positions: vec![Position { x: 1, y: 1 }],
                side: Players::Blank
            })
        );
    }

    #[test]
    fn expected_loss() {
        let mut board = vec![vec![Players::Blank; 3]; 3];
        board[0][0] = Players::PlayerX;
        board[0][2] = Players::PlayerO;
        board[1][0] = Players::PlayerX;
        board[2][0] = Players::PlayerO;
        board[2][2] = Players::PlayerO;
        let responses = minimax(Players::PlayerX, &board);
        assert_eq!(
            responses,
            Some(PlayActions {
                positions: vec![
                    Position { x: 1, y: 0 },
                    Position { x: 1, y: 1 },
                    Position { x: 2, y: 1 },
                    Position { x: 1, y: 2 }
                ],
                side: Players::PlayerO
            })
        );
    }
}

fn main() {}
```