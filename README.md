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

			if(dribble_init_flag==1 && Allowed_Flag==1)
			{
				Unitree_Set_Parametr(&Sunagch_Send, 1, 0.75, 0,-2, 4, 0.01);
				Sunagch_old=Sunagch_Send;
				SERVO_Send_recv(&Sunagch_Send, &Sunagch_Recv);
				if(SUNAGH_AMT102_pos<=0)
				{
					HAL_GPIO_WritePin(BALL_HYDRAULIC_GPIO_Port,BALL_HYDRAULIC_Pin,1);
					vTaskDelay(250);
					HAL_GPIO_WritePin(BALL_HYDRAULIC_GPIO_Port,BALL_HYDRAULIC_Pin,0);
					osDelay(750);
					if(HAL_GPIO_ReadPin(IR_SENSOR_GPIO_Port,IR_SENSOR_Pin)==0)
					{
						Unitree_Set_Parametr(&Elbow_Send, 1, -0.5, 0, Elbow_old.Pos, 0.8, 0.08);
						move_elbow_smoothly(Elbow_old.Pos, ELBOW_DUNK_POS);
						osDelay(100);
						Unitree_Set_Parametr(&Sunagch_Send, 1, 0, 0, SUNAGCH_PASS_POS, 0.2, 0.1);
						Sunagch_old=Sunagch_Send;
						SERVO_Send_recv(&Sunagch_Send, &Sunagch_Recv);
					}
					Allowed_Flag=0;
					Robot_Status=0;
					dribble_init_flag=0;
					vTaskSuspend(NULL);
				}
			}



	for(;;)
	{
		if(Robot_Status==Dunk)
		{
			if(dunk_init_flag==1 && Allowed_Flag==1)
			{
				Unitree_Set_Parametr(&Sunagch_Send, 1, -2.5, 0, -80, 1, 0.02);
				SERVO_Send_recv(&Sunagch_Send, &Sunagch_Recv);
				osDelay(5);
				Unitree_Set_Parametr(&Right_Leg_Send, 1, 0.5, 0, M1_init, 14, 0.02);
				Unitree_Set_Parametr(&Left_Leg_Send, 1, -0.5, 0, M2_Init, 14, 0.02);
				SERVO_Send_recv(&Left_Leg_Send, &Left_Leg_Recv);
				osDelay(5);
				SERVO_Send_recv(&Right_Leg_Send, &Right_Leg_Recv);
				osDelay(30);
				Unitree_Set_Parametr(&Right_Leg_Send, 1, 3, 0, M1_init + 16, 18, 0.01);
				Unitree_Set_Parametr(&Left_Leg_Send, 1, -3, 0, M2_Init - 16, 18, 0.01);
				SERVO_Send_recv(&Left_Leg_Send, &Left_Leg_Recv);
				osDelay(5);
				SERVO_Send_recv(&Right_Leg_Send, &Right_Leg_Recv);
				if(SUNAGH_AMT102_pos<=-50)
				{
					Unitree_Set_Parametr(&Right_Leg_Send, 1, -0.5, 0, M1_init + 3, 7, 0.02);
					Unitree_Set_Parametr(&Left_Leg_Send, 1, 0.5, 0, M2_Init - 3, 7, 0.02);
					SERVO_Send_recv(&Right_Leg_Send, &Right_Leg_Recv);
					osDelay(5);
					SERVO_Send_recv(&Left_Leg_Send, &Left_Leg_Recv);
					osDelay(25);
					HAL_GPIO_WritePin(BALL_HYDRAULIC_GPIO_Port,BALL_HYDRAULIC_Pin,1);
					osDelay(80);
					Unitree_Set_Parametr(&Sunagch_Send, 1, 0.5, 0,-65, 2, 0.06	);
					SERVO_Send_recv(&Sunagch_Send, &Sunagch_Recv);
					osDelay(1500);
					jump_flag = 0;
					Robot_Status=0;
					dunk_init_flag=Allowed_Flag=0;
					jump_state = 0;
					osDelay(50);
					HAL_GPIO_WritePin(BALL_HYDRAULIC_GPIO_Port,BALL_HYDRAULIC_Pin,0);
					Unitree_Set_Parametr(&Right_Leg_Send, 1, 0, 0, M1_init, 2 , 0.08);
					Unitree_Set_Parametr(&Left_Leg_Send, 1, 0, 0, M2_Init, 2, 0.08);
					SERVO_Send_recv(&Right_Leg_Send, &Right_Leg_Recv);
					vTaskDelay(20);
					SERVO_Send_recv(&Left_Leg_Send, &Left_Leg_Recv);
					vTaskSuspend(NULL);
				}
				//				switch (jump_state)
				//				{
				//				case 1	:
				//					Unitree_Set_Parametr(&Sunagch_Send, 1, -1.25, 0, -45, 0.5, 0.05);
				//					SERVO_Send_recv(&Sunagch_Send, &Sunagch_Recv);
				//					jump_start_time=osKernelGetTickCount();
				//					jump_state=2;
				//					break;
				//				case 2:
				//					//if (osKernelGetTickCount() - jump_start_time > 60)
				//					if (osKernelGetTickCount() - jump_start_time > 90)
				//					{
				//						Unitree_Set_Parametr(&Right_Leg_Send, 1, 0.5, 0, M1_init, 14, 0.02);
				//						Unitree_Set_Parametr(&Left_Leg_Send, 1, -0.5, 0, M2_Init, 14, 0.02);
				//						SERVO_Send_recv(&Left_Leg_Send, &Left_Leg_Recv);
				//						osDelay(5);
				//						SERVO_Send_recv(&Right_Leg_Send, &Right_Leg_Recv);
				//						jump_start_time = osKernelGetTickCount();
				//						jump_state = 3;
				//					}
				//					break;
				//				case 3:
				//					if (osKernelGetTickCount() - jump_start_time > 30	)
				//					{
				//						Unitree_Set_Parametr(&Right_Leg_Send, 1, 1.2, 0, M1_init + 18, 17, 0.01);
				//						Unitree_Set_Parametr(&Left_Leg_Send, 1, -1.2, 0, M2_Init - 18, 17, 0.01);
				//						SERVO_Send_recv(&Left_Leg_Send, &Left_Leg_Recv);
				//						osDelay(5);
				//						SERVO_Send_recv(&Right_Leg_Send, &Right_Leg_Recv);
				//						jump_start_time = osKernelGetTickCount();
				//						jump_state = 4;
				//					}
				//					break;
				//				case 4:
				//					if (osKernelGetTickCount() - jump_start_time > 220)
				//					{
				//						Unitree_Set_Parametr(&Right_Leg_Send, 1, -0.5, 0, M1_init + 3, 7, 0.02);
				//						Unitree_Set_Parametr(&Left_Leg_Send, 1, 0.5, 0, M2_Init - 3, 7, 0.02);
				//						Unitree_Set_Parametr(&Sunagch_Send, 1, -1.5, 0,-79, 6, 0.04	);
				//						SERVO_Send_recv(&Sunagch_Send, &Sunagch_Recv);
				//						osDelay(10);
				//						SERVO_Send_recv(&Right_Leg_Send, &Right_Leg_Recv);
				//						osDelay(5);
				//						SERVO_Send_recv(&Left_Leg_Send, &Left_Leg_Recv);
				//						osDelay(40);
				//						HAL_GPIO_WritePin(BALL_HYDRAULIC_GPIO_Port,BALL_HYDRAULIC_Pin,1);
				//						osDelay(50);
				//						Unitree_Set_Parametr(&Sunagch_Send, 1, 0.5, 0,-60, 2, 0.06	);
				//						SERVO_Send_recv(&Sunagch_Send, &Sunagch_Recv);
				//						jump_start_time = osKernelGetTickCount();
				//
				//						jump_state = 5;
				//					}
				//					break;
				//				case 5:
				//					if (osKernelGetTickCount() - jump_start_time > 1500)
				//					{
				//
				//						jump_flag = 0;
				//						Robot_Status=0;
				//						dunk_init_flag=Allowed_Flag=0;
				//						jump_state = 0;
				//						osDelay(50);
				//						HAL_GPIO_WritePin(BALL_HYDRAULIC_GPIO_Port,BALL_HYDRAULIC_Pin,0);
				//						Unitree_Set_Parametr(&Right_Leg_Send, 1, 0, 0, M1_init, 2 , 0.08);
				//						Unitree_Set_Parametr(&Left_Leg_Send, 1, 0, 0, M2_Init, 2, 0.08);
				//						SERVO_Send_recv(&Right_Leg_Send, &Right_Leg_Recv);
				//						vTaskDelay(20);
				//						SERVO_Send_recv(&Left_Leg_Send, &Left_Leg_Recv);
				//						vTaskSuspend(NULL);
				//					}
				//					break;
				//
				//				}
			}
			else if(dunk_init_flag==0)
			{
				jump_state = 1;
				Unitree_Set_Parametr(&Right_Leg_Send, 1, 0, 0, M1_init - 0.5, 2, 0.02);
				Unitree_Set_Parametr(&Left_Leg_Send, 1, 0, 0, M2_Init + 0.5, 2, 0.02);
				//				SERVO_Send_recv(&Right_Leg_Send, &Right_Leg_Recv);
				//				osDelay(10);
				//				SERVO_Send_recv(&Left_Leg_Send, &Left_Leg_Recv);
				HAL_GPIO_WritePin(BALL_HYDRAULIC_GPIO_Port,BALL_HYDRAULIC_Pin,0);
				Unitree_Set_Parametr(&Sunagch_Send, 1, 0, 0,-20, 0.2, 0.02);
				SERVO_Send_recv(&Sunagch_Send, &Sunagch_Recv);
				Unitree_Set_Parametr(&Elbow_Send, 1, -1, 0,Elbow_old.Pos, 0.4, 0.1);
				move_elbow_smoothly(Elbow_old.Pos,ELBOW_DUNK_POS);
				arm_angle_pid.SP=-15;
				dunk_init_flag=task_init_flag=1;
				jump_start_time = osKernelGetTickCount();
			}
		}
		else
		{
			dunk_init_flag=task_init_flag=0;
			vTaskSuspend(NULL);
		}
		osDelay(10);
	}
	/* USER CODE END Dunk_task */
}





















		if(Robot_Status==Dunk)
		{
			if(dunk_init_flag==1 && Allowed_Flag==1)
			{
				if(jump_state==1)
				{
					Unitree_Set_Parametr(&Sunagch_Send, 1, -4.5, 0, -30, 5, 0.01);
					SERVO_Send_recv(&Sunagch_Send, &Sunagch_Recv);
					jump_state=2;
				}
				if(SUNAGCH_AMT102_POS<-20 && jump_state==2)
				{
					//					Unitree_Set_Parametr(&Right_Leg_Send, 1, 0.5, 0, M1_init, 14, 0.01);
					//					Unitree_Set_Parametr(&Left_Leg_Send, 1, -0.5, 0, M2_Init, 14, 0.01);
					//					SERVO_Send_recv(&Left_Leg_Send, &Left_Leg_Recv);
					//					osDelay(5);
					//					SERVO_Send_recv(&Right_Leg_Send, &Right_Leg_Recv);
					//					osDelay(30);
					Unitree_Set_Parametr(&Right_Leg_Send, 1, 3, 0, M1_init + 17, 18, 0.01);
					Unitree_Set_Parametr(&Left_Leg_Send, 1, -3, 0, M2_Init - 17, 18, 0.01);
					SERVO_Send_recv(&Left_Leg_Send, &Left_Leg_Recv);
					osDelay(5);
					SERVO_Send_recv(&Right_Leg_Send, &Right_Leg_Recv);
					osDelay(5);
					osDelay(130);
					Unitree_Set_Parametr(&Sunagch_Send, 1, -4, 0, -81, 5, 0.01);
					SERVO_Send_recv(&Sunagch_Send, &Sunagch_Recv);
					osDelay(125);
					Unitree_Set_Parametr(&Right_Leg_Send, 1, -0.5, 0, M1_init + 3, 7, 0.02);
					Unitree_Set_Parametr(&Left_Leg_Send, 1, 0.5, 0, M2_Init - 3, 7, 0.02);
					SERVO_Send_recv(&Right_Leg_Send, &Right_Leg_Recv);
					osDelay(5);
					SERVO_Send_recv(&Left_Leg_Send, &Left_Leg_Recv);
					jump_state=3;
				}
				if(jump_state==3)
				{
					if(SUNAGCH_AMT102_POS<-50)
					{
						HAL_GPIO_WritePin(BALL_HYDRAULIC_GPIO_Port,BALL_HYDRAULIC_Pin,1);
						osDelay(80);
						jump_state=4;
					}
				}
				if(jump_state==4)
				{
					if(SUNAGCH_AMT102_POS<-80)
					{
						osDelay(100);
						Unitree_Set_Parametr(&Sunagch_Send, 1, 0.5, 0,-65, 2, 0.06	);
						SERVO_Send_recv(&Sunagch_Send, &Sunagch_Recv);
						osDelay(2000);
						jump_flag = 0;
						Robot_Status=0;
						dunk_init_flag=Allowed_Flag=0;
						osDelay(50);
						HAL_GPIO_WritePin(BALL_HYDRAULIC_GPIO_Port,BALL_HYDRAULIC_Pin,0);
						Unitree_Set_Parametr(&Right_Leg_Send, 1, 0, 0, M1_init, 2 , 0.08);
						Unitree_Set_Parametr(&Left_Leg_Send, 1, 0, 0, M2_Init, 2, 0.08);
						SERVO_Send_recv(&Right_Leg_Send, &Right_Leg_Recv);
						vTaskDelay(20);
						SERVO_Send_recv(&Left_Leg_Send, &Left_Leg_Recv);
						jump_state=0;
						vTaskSuspend(NULL);
					}
				}

			}
			else if(dunk_init_flag==0)
			{
				jump_state = 1;
				HAL_GPIO_WritePin(BALL_HYDRAULIC_GPIO_Port,BALL_HYDRAULIC_Pin,0);
				Unitree_Set_Parametr(&Sunagch_Send, 1, 0, 0,-10, 0.2, 0.02);
				SERVO_Send_recv(&Sunagch_Send, &Sunagch_Recv);
				Unitree_Set_Parametr(&Elbow_Send, 1, -1, 0,Elbow_old.Pos, 0.4, 0.1);
				move_elbow_smoothly(Elbow_old.Pos,ELBOW_DUNK_POS);
				arm_angle_pid.SP=15;
				dunk_init_flag=task_init_flag=1;
				jump_start_time = osKernelGetTickCount();
			}
		}
		else
		{
			dunk_init_flag=task_init_flag=0;
			vTaskSuspend(NULL);
		}
		osDelay(10);
	}
	/* USER CODE END Dunk_task */
}
