import cv2
import mediapipe
import time
import random


def calculate_game_state(move):
    moves = ["Rock", "Paper", "Scissors"]
    wins = {"Rock": "Scissors", "Paper": "Rock", "Scissors": "Paper"}
    selected = random.randint(0, 2)
    print("Computer played " + moves[selected])

    if moves[selected] == move:
        return 0, moves[selected]
    if wins[move] == moves[selected]:
        return 1, moves[selected]
    return -1, moves[selected]


def determine_winner(p1_move, p2_move):
    if p1_move == p2_move:
        return 0
    wins = {"Rock": "Scissors", "Paper": "Rock", "Scissors": "Paper"}
    if wins[p1_move] == p2_move:
        return 1
    return -1


def get_finger_status(hands_module, hand_landmarks, finger_name):
    finger_id_map = {'INDEX': 8, 'MIDDLE': 12, 'RING': 16, 'PINKY': 20}
    tip_y = hand_landmarks.landmark[finger_id_map[finger_name]].y
    pip_y = hand_landmarks.landmark[finger_id_map[finger_name] - 2].y
    return tip_y < pip_y


def get_move_from_landmarks(hands_module, hand_landmarks):
    index = get_finger_status(hands_module, hand_landmarks, 'INDEX')
    middle = get_finger_status(hands_module, hand_landmarks, 'MIDDLE')
    ring = get_finger_status(hands_module, hand_landmarks, 'RING')
    pinky = get_finger_status(hands_module, hand_landmarks, 'PINKY')

    if not index and not middle and not ring and not pinky:
        return "Rock"
    if index and middle and ring and pinky:
        return "Paper"
    if index and middle and not ring and not pinky:
        return "Scissors"
    return "UNKNOWN"


def start_video():
    drawing = mediapipe.solutions.drawing_utils
    hands_module = mediapipe.solutions.hands

    cap = cv2.VideoCapture(0)
    start_time = 0
    timer_started = False
    time_left = 3
    hold_play = False
    game_over_text = ""
    computer_played = ""

    with hands_module.Hands(
            static_image_mode=False,
            min_detection_confidence=0.7,
            min_tracking_confidence=0.4,
            max_num_hands=1) as hands:
        while True:
            if timer_started:
                now = time.time()
                if now - start_time >= 1:
                    time_left -= 1
                    start_time = now
                    if time_left <= 0:
                        hold_play = True
                        timer_started = False

            ret, frame = cap.read()
            if not ret:
                break

            frame_rgb = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)
            results = hands.process(frame_rgb)

            move = "UNKNOWN"
            if results.multi_hand_landmarks:
                for hand_landmarks in results.multi_hand_landmarks:
                    drawing.draw_landmarks(frame, hand_landmarks, hands_module.HAND_CONNECTIONS)
                    move = get_move_from_landmarks(hands_module, hand_landmarks)

                    if hold_play and move != "UNKNOWN":
                        hold_play = False
                        print("Player played", move)
                        result, comp_move = calculate_game_state(move)
                        computer_played = f"You: {move} | Computer: {comp_move}"
                        if result == 1:
                            game_over_text = "You Win!"
                        elif result == -1:
                            game_over_text = "You Lose!"
                        else:
                            game_over_text = "Draw!"

            font = cv2.FONT_HERSHEY_SIMPLEX
            if not hold_play and not timer_started:
                cv2.putText(frame, game_over_text + " " + computer_played, (10, 450),
                            font, 0.75, (255, 255, 255), 2)

            if hold_play:
                label = "PLAY NOW!"
            elif timer_started:
                label = f"STARTING IN {time_left}"
            else:
                label = "PRESS SPACE TO START | Q TO QUIT"

            cv2.putText(frame, label, (50, 50), font, 1, (0, 255, 255), 2)

            cv2.imshow("RPS vs AI", frame)

            key = cv2.waitKey(1)
            if key == 32:  # Space
                start_time = time.time()
                time_left = 3
                timer_started = True
            elif key == ord('q') or key == 27:  # Q or ESC
                break

    cap.release()
    cv2.destroyAllWindows()


def start_video_two_players():
    drawing = mediapipe.solutions.drawing_utils
    hands_module = mediapipe.solutions.hands

    cap = cv2.VideoCapture(0)
    start_time = 0
    timer_started = False
    time_left = 3
    hold_play = False
    game_over_text = ""
    result_text = ""

    with hands_module.Hands(
            static_image_mode=False,
            min_detection_confidence=0.7,
            min_tracking_confidence=0.4,
            max_num_hands=2) as hands:
        while True:
            if timer_started:
                now = time.time()
                if now - start_time >= 1:
                    time_left -= 1
                    start_time = now
                    if time_left <= 0:
                        hold_play = True
                        timer_started = False

            ret, frame = cap.read()
            if not ret:
                break

            frame_rgb = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)
            results = hands.process(frame_rgb)

            p1_move = "UNKNOWN"
            p2_move = "UNKNOWN"

            if results.multi_hand_landmarks and len(results.multi_hand_landmarks) >= 2:
                hands_list = results.multi_hand_landmarks[:2]
                p1_move = get_move_from_landmarks(hands_module, hands_list[0])
                p2_move = get_move_from_landmarks(hands_module, hands_list[1])

                for hand in hands_list:
                    drawing.draw_landmarks(frame, hand, hands_module.HAND_CONNECTIONS)

                if hold_play and p1_move != "UNKNOWN" and p2_move != "UNKNOWN":
                    hold_play = False
                    result = determine_winner(p1_move, p2_move)
                    if result == 1:
                        game_over_text = "Player 1 Wins!"
                    elif result == -1:
                        game_over_text = "Player 2 Wins!"
                    else:
                        game_over_text = "Draw!"
                    result_text = f"P1: {p1_move} | P2: {p2_move}"

            font = cv2.FONT_HERSHEY_SIMPLEX
            if not hold_play and not timer_started:
                cv2.putText(frame, game_over_text + " " + result_text, (10, 450),
                            font, 0.75, (255, 255, 255), 2)

            if hold_play:
                label = "PLAY NOW!"
            elif timer_started:
                label = f"STARTING IN {time_left}"
            else:
                label = "PRESS SPACE TO START | Q TO QUIT"

            cv2.putText(frame, label, (50, 50), font, 1, (0, 255, 255), 2)
            cv2.imshow("RPS 2 Players", frame)

            key = cv2.waitKey(1)
            if key == 32:
                start_time = time.time()
                time_left = 3
                timer_started = True
            elif key == ord('q') or key == 27:
                break

    cap.release()
    cv2.destroyAllWindows()


def main():
    while True:
        choice = input("Choose game mode:\n1: Play vs AI\n2: Play 2 Players\nQ: Quit\nEnter choice: ").strip().lower()
        if choice == "1":
            start_video()
        elif choice == "2":
            start_video_two_players()
        elif choice == "q":
            break
        else:
            print("Invalid input. Try again.")


if __name__ == "__main__":
    main()
