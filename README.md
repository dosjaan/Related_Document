		float pivot_blend = (fabsf(JOY.LY) < 0.1f) ? 0.7f : 0.3f;
		if (fabsf(JOY.LY) < joystick_deadzone) JOY.LY = 0;
		if (fabsf(JOY.RX) < joystick_deadzone) JOY.RX = 0;

		float rotation_scale = 0.4f * (1.0f - fabsf(JOY.LY)) + 0.25f;
		float left_wheel = JOY.LY*(1-pivot_blend) + JOY.RX * rotation_scale*pivot_blend;
		float right_wheel = JOY.LY*(1-pivot_blend) - JOY.RX * rotation_scale*pivot_blend;
		float max_speed = fmaxf(fabsf(left_wheel), fabsf(right_wheel));
		if (max_speed > 1.0f)
		{
			left_wheel /= max_speed;
			right_wheel /= max_speed;
		}
		float target_left_speed = left_wheel * Diff_MAX_SPEED;
		float target_right_speed = right_wheel * Diff_MAX_SPEED;

		// Smooth filter (limits sudden change)
		current_speed_left = current_speed_left * (1.0f - SMOOTHING_FACTOR) + target_left_speed * SMOOTHING_FACTOR;
		current_speed_right = current_speed_right * (1.0f - SMOOTHING_FACTOR) + target_right_speed * SMOOTHING_FACTOR;

		// Clamp final speeds to safe range (optional guard)
		if (fabsf(current_speed_left) > Diff_MAX_SPEED) current_speed_left = (current_speed_left > 0) ? Diff_MAX_SPEED : -Diff_MAX_SPEED;
		if (fabsf(current_speed_right) > Diff_MAX_SPEED) current_speed_right = (current_speed_right > 0) ? Diff_MAX_SPEED : -Diff_MAX_SPEED;

		// Send to motor
		setVelocity(&huart2, 0, current_speed_right, 0.0f);
		setVelocity(&huart2, 1, -current_speed_left, 0.0f);
		osDelay(10);
