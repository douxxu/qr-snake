# qr-snake
Hey, wanna play a simple snake game in your shell ? But where is the code ? It's easy. It's just here:

![snake](/snake.png)

Yeah it's a pretty big qr code, there are __1888 bytes__ (1.8kb) stored in ! It's the size of the optimized code.
You can use [zbarimg](https://github.com/mchehab/zbar) to extact the code and play it!

## Run snake
1. Download the qr code
2. If it's your first time, use
```
zbarimg -q snake.png | awk 'NR==1{print substr($0, 9)} NR>1{print}' > snake && chmod +x snake && ./snake
```
If you already did that, simply do 
```
./snake
```

<details>
  <summary>Human readable code</summary>
  
```bash
#!/bin/bash

# Initialize variables
score=0                  # Score
width=30                 # Width of the field
height=15                # Height of the field
snake_x=(10 9 8)         # Snake's x-coordinates
snake_y=(7 7 7)          # Snake's y-coordinates
direction=R              # Current direction
fruit_x=$((RANDOM % width))  # Fruit's x-position
fruit_y=$((RANDOM % height)) # Fruit's y-position

# Function to update the snake's position
move_snake() {
  local new_x=${snake_x[0]}      # New x-position
  local new_y=${snake_y[0]}      # New y-position

  # Update position based on direction
  case $direction in
    R) ((new_x++)) ;;
    L) ((new_x--)) ;;
    U) ((new_y--)) ;;
    D) ((new_y++)) ;;
  esac

  # Handle border crossing
  if ((new_x >= width)); then new_x=0; fi
  if ((new_x < 0)); then new_x=$((width - 1)); fi
  if ((new_y >= height)); then new_y=0; fi
  if ((new_y < 0)); then new_y=$((height - 1)); fi

  # Check for collision with the snake's body
  for ((i=1; i<${#snake_x[@]}; i++)); do
    if [[ ${snake_x[i]} -eq $new_x && ${snake_y[i]} -eq $new_y ]]; then
      echo "Game Over! Score: $score"
      exit
    fi
  done

  # Update the snake's coordinates
  snake_x=($new_x "${snake_x[@]}")
  snake_y=($new_y "${snake_y[@]}")
  snake_x=("${snake_x[@]:0:${#snake_x[@]}-1}")
  snake_y=("${snake_y[@]:0:${#snake_y[@]}-1}")
}

# Function to display the field
display_field() {
  clear
  for ((j=0; j<height; j++)); do
    for ((i=0; i<width; i++)); do
      local drawn=false

      # Display the snake's head
      if [[ $i -eq ${snake_x[0]} && $j -eq ${snake_y[0]} ]]; then
        echo -n "0"
        drawn=true
      else
        # Display the snake's body
        for ((k=1; k<${#snake_x[@]}; k++)); do
          if [[ $i -eq ${snake_x[k]} && $j -eq ${snake_y[k]} ]]; then
            echo -n "o"
            drawn=true
            break
          fi
        done
      fi

      # Display the fruit or empty space
      if [[ $drawn == false ]]; then
        if [[ $i -eq $fruit_x && $j -eq $fruit_y ]]; then
          echo -n "@"
        else
          echo -n "."
        fi
      fi
    done
    echo
  done
  echo "Score: $score"
}

# Function to generate a new fruit position
generate_fruit() {
  while true; do
    fruit_x=$((RANDOM % width))
    fruit_y=$((RANDOM % height))
    local collision=false
    for ((i=0; i<${#snake_x[@]}; i++)); do
      if [[ ${snake_x[i]} -eq $fruit_x && ${snake_y[i]} -eq $fruit_y ]]; then
        collision=true
        break
      fi
    done
    if [[ $collision == false ]]; then break; fi
  done
}

# Main game loop
while true; do
  read -n 1 -t 0.1 key
  case $key in
    w) [[ $direction != D ]] && direction=U ;;
    s) [[ $direction != U ]] && direction=D ;;
    a) [[ $direction != R ]] && direction=L ;;
    d) [[ $direction != L ]] && direction=R ;;
    q) exit ;;
  esac

  move_snake
  if [[ ${snake_x[0]} -eq $fruit_x && ${snake_y[0]} -eq $fruit_y ]]; then
    snake_x+=(${snake_x[-1]})
    snake_y+=(${snake_y[-1]})
    score=$((score + 1))
    generate_fruit
  fi
  display_field
done

```
</details>
