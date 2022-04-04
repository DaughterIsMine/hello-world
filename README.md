# hello-world

import numpy as np
from matplotlib import pyplot as plt

def coupler_2_2(lo_form, signal_form):
    coupler_matrix = np.array([[1 / np.sqrt(2), 1 / np.sqrt(2) * np.exp(1j * np.pi / 2)],
                               [1 / np.sqrt(2) * np.exp(1j * np.pi / 2), 1 / np.sqrt(2)]])

# shot noise by photon
def shot_noise(optical_power, responsivity, bandwidth, source_length, quantum = False, optical_frequency = 1550, quantum_efficiency = 1):
    h = 6.626e-34   # Planck constant
    e = 1.602e-19   # electron charge [C] = [A] x [sec]

    if quantum:
        mean_photon_flux = optical_power / (h * optical_frequency)
        mean_photon_electron = quantum_efficiency * mean_photon_flux
        mean_photo_current = np.sqrt(2 * e * bandwidth * mean_photon_electron)
    elif not quantum:
        mean_photo_current = np.sqrt(2 * e * responsivity * optical_power * bandwidth)

    shot_noise_variable = np.random.normal(mean_photo_current, mean_photo_current, source_length) # due to central limit theorem (normal = poisson)

    return shot_noise_variable, mean_photo_current

# thermal noise by resister
def thermal_noise(temperature, resistance, bandwidth, source_length):
    k = 1.381e-23   # Boltzmann constant
    absolute_temperature = temperature + 273.15

    thermal_variance = 4 * k * absolute_temperature * bandwidth / resistance

    thermal_noise_variable = np.random.normal(0, np.sqrt(thermal_variance), source_length)

    return thermal_noise_variable, np.sqrt(thermal_variance)

# dark_current_noise
def dark_current_noise(bandwidth, source_length, material = 'ingaas'):
    e = 1.602e-19  # electron charge [C] = [A] x [sec]

    if material == 'ingaas':
        dark_current = 1e-9

    mean_dark_current = np.sqrt(2 * e * dark_current * bandwidth)

    dark_noise = np.random.normal(mean_dark_current, mean_dark_current, source_length) # due to central limit theorem (normal = poisson)

    return dark_noise, mean_dark_current

# background_noise
def background_noise(bandwidth, source_length, material = 'ingaas'):
    e = 1.602e-19  # electron charge [C] = [A] x [sec]

    if material == 'ingaas':
        dark_current = 1e-9

    mean_dark_current = np.sqrt(2 * e * dark_current * bandwidth)

    dark_noise = np.random.normal(mean_dark_current, mean_dark_current, source_length) # due to central limit theorem (normal = poisson)

    return dark_noise

if __name__ == "__main__":
    responsivity = 1
    optical_power = 1
    c = 3e8
    center_wavelength = 1550e-9
    optical_frequency = c / center_wavelength
    bandwidth = 2500000
    source_length = 100 * 10
    resistance = 50
    temperature = 25
    sampling_frequency = 500e6

    x = shot_noise(optical_power * 100, responsivity, bandwidth, source_length)[0]

    plt.figure()
    plt.subplot(2,1,1)
    plt.plot(shot_noise(optical_power * 100, responsivity, bandwidth, source_length)[0] +
             shot_noise(optical_power, responsivity, bandwidth, source_length)[0] + dark_current_noise(bandwidth, source_length)[0])
    plt.subplot(2,1,2)
    plt.plot(thermal_noise(temperature, resistance, bandwidth, source_length)[0])


    fft_result = np.fft.fft(x) / source_length
    fft_result = fft_result[0:int(source_length/ 2)]
    frequency_range = sampling_frequency / 2 * np.linspace(0, 1, int(source_length / 2))

    plt.figure()
    plt.plot(frequency_range[1:], abs(fft_result[1:]))
    plt.title('fft result')
    plt.xlabel('frequency[Hz]')


    plt.show()













Just introduction of git-hub



Hi, I'm Ingyu Jang!


import numpy as np
from matplotlib import pyplot as plt

c = 3e8                                 # light speed
wavelength = 1550e-9                    # wavelength
del_wavelength = 2e-9                   # wavelength range
f1 = c / wavelength                     # max frequency
f0 = c / (wavelength + del_wavelength)  # min frequency
T = 2e-3                                # period
distance = 250                          # target distance
tau = distance / c                      # target distance / light speed
r = 0.1                                 # target reflectance
E = 1                                   # light power
fs = 500e6                              # sampling frequency

time = np.arange(0, T, 1 / fs)
time_length = len(time)

Emission = E * np.exp(1j * 2 * np.pi * (f0 + (f1 - f0) / T * time) * time)
Reflection = r * E * np.exp(1j * 2 * np.pi * (f0 + (f1 - f0) / T * (time - tau)) * (time - tau))

Addition = Emission + Reflection

Intensity = np.abs(Addition)

fft_result = np.fft.fft(Intensity)
fft_result = fft_result[0:int(time_length / 2)]
frequency_range = fs / 2 * np.linspace(0, 1, int(time_length / 2))

print('beat frequency =', frequency_range[np.argmax(abs(fft_result[1:])) + 1], 'Hz')
plt.plot(frequency_range[1:], abs(fft_result[1:]))
plt.show()




#include <windows.h>
#include "stdio.h"
#include "start.h"
#include <math.h>

///////////////////////////////////////////////////////////////////////////////////////////////////////////////


LRESULT CALLBACK WndProc(HWND,UINT,WPARAM,LPARAM);
HINSTANCE g_hInst;
LPSTR lpszClass="TOF simulator";

int APIENTRY WinMain(HINSTANCE hInstance,HINSTANCE hPrevInstance
		  ,LPSTR lpszCmdParam,int nCmdShow)
{
	HWND hWnd;
	MSG Message;
	WNDCLASS WndClass;
	g_hInst=hInstance;
	
	WndClass.cbClsExtra=0;
	WndClass.cbWndExtra=0;
	WndClass.hbrBackground=(HBRUSH)GetStockObject(WHITE_BRUSH);
	WndClass.hCursor=LoadCursor(NULL,IDC_ARROW);
	WndClass.hIcon=LoadIcon(NULL,IDI_APPLICATION);
	WndClass.hInstance=hInstance;
	WndClass.lpfnWndProc=(WNDPROC)WndProc;
	WndClass.lpszClassName=lpszClass;
	WndClass.lpszMenuName=NULL;
	WndClass.style=CS_HREDRAW | CS_VREDRAW;
	RegisterClass(&WndClass);

	hWnd=CreateWindow(lpszClass,lpszClass,WS_OVERLAPPEDWINDOW,
		0,0,1920,1080,
		NULL,(HMENU)NULL,hInstance,NULL);
	/*hWnd=CreateWindow(lpszClass,lpszClass,WS_OVERLAPPEDWINDOW,
		CW_USEDEFAULT,CW_USEDEFAULT,CW_USEDEFAULT,CW_USEDEFAULT,
		NULL,(HMENU)NULL,hInstance,NULL);*/
	ShowWindow(hWnd,nCmdShow);

	while(GetMessage(&Message,0,0,0)) {
		TranslateMessage(&Message);
		DispatchMessage(&Message);
	}
	return Message.wParam;
}

#include <math.h>
LRESULT CALLBACK WndProc(HWND hWnd,UINT iMessage,WPARAM wParam,LPARAM lParam)
{
	HDC hdc;
	PAINTSTRUCT ps;
	double f;
	int y;

	HPEN MyPen, OldPen;
	char buf[100];
	RECT rt;

	switch (iMessage) {

	case WM_CREATE:
	{
		SetTimer(hWnd, 0, 50, NULL);

		//버튼
		h_ChartRef95 = CreateWindow("button", "차트반사율93%", WS_CHILD | WS_VISIBLE | BS_PUSHBUTTON,
			100, 650, 120, 30, hWnd, (HMENU)101, hInst, NULL);
		h_ChartRef50 = CreateWindow("button", "차트반사율50%", WS_CHILD | WS_VISIBLE | BS_PUSHBUTTON,
			230, 650, 120, 30, hWnd, (HMENU)102, hInst, NULL);
		h_ChartRef10 = CreateWindow("button", "차트반사율10%", WS_CHILD | WS_VISIBLE | BS_PUSHBUTTON,
			360, 650, 120, 30, hWnd, (HMENU)103, hInst, NULL);

		h_Load = CreateWindow("button", "설정값 적용", WS_CHILD | WS_VISIBLE | BS_PUSHBUTTON,
			50, 10, 120, 30, hWnd, (HMENU)104, hInst, NULL);
		h_Txmode = CreateWindow("button", "Flood/Dot", WS_CHILD | WS_VISIBLE | BS_PUSHBUTTON,
			50, 50, 120, 30, hWnd, (HMENU)105, hInst, NULL);
		h_AreaImage = CreateWindow("button", "Area 계산", WS_CHILD | WS_VISIBLE | BS_PUSHBUTTON,
			50 + 125, 10, 120, 30, hWnd, (HMENU)106, hInst, NULL);
		h_Outdoors = CreateWindow("button", "In/Out door", WS_CHILD | WS_VISIBLE | BS_PUSHBUTTON,
			50+125, 50, 120, 30, hWnd, (HMENU)107, hInst, NULL);
		B_Save = CreateWindow("button", "Save", WS_CHILD | WS_VISIBLE | BS_PUSHBUTTON,
			50, 90, 120*2+5, 30*2, hWnd, (HMENU)108, hInst, NULL);

		h_OpenFile1 = CreateWindow("button", "Sensor & TX설정", WS_CHILD | WS_VISIBLE | BS_PUSHBUTTON,
			1500, 10, 120, 30, hWnd, (HMENU)121, hInst, NULL);
		h_OpenFile2 = CreateWindow("button", "Rx lens 설정", WS_CHILD | WS_VISIBLE | BS_PUSHBUTTON,
			1500+125, 10, 120, 30, hWnd, (HMENU)122, hInst, NULL);
		h_OpenFile3 = CreateWindow("button", "VCSEL 설정", WS_CHILD | WS_VISIBLE | BS_PUSHBUTTON,
			1500, 10+35, 120, 30, hWnd, (HMENU)123, hInst, NULL);
		h_OpenFile4 = CreateWindow("button", "DOE 설정", WS_CHILD | WS_VISIBLE | BS_PUSHBUTTON,
			1500+125, 10+35, 120, 30, hWnd, (HMENU)124, hInst, NULL);

		h_ChartRef10 = CreateWindow("button", "0.1F%", WS_CHILD | WS_VISIBLE | BS_PUSHBUTTON,
			100, 700, 50, 40, hWnd, (HMENU)111, hInst, NULL);
		h_ChartRef10 = CreateWindow("button", "0.2F%", WS_CHILD | WS_VISIBLE | BS_PUSHBUTTON,
			160, 700, 50, 40, hWnd, (HMENU)112, hInst, NULL);
		h_ChartRef10 = CreateWindow("button", "0.3F%", WS_CHILD | WS_VISIBLE | BS_PUSHBUTTON,
			220, 700, 50, 40, hWnd, (HMENU)113, hInst, NULL);
		h_ChartRef10 = CreateWindow("button", "0.4F%", WS_CHILD | WS_VISIBLE | BS_PUSHBUTTON,
			280, 700, 50, 40, hWnd, (HMENU)114, hInst, NULL);
		h_ChartRef10 = CreateWindow("button", "0.5F%", WS_CHILD | WS_VISIBLE | BS_PUSHBUTTON,
			340, 700, 50, 40, hWnd, (HMENU)115, hInst, NULL);
		h_ChartRef10 = CreateWindow("button", "0.6F%", WS_CHILD | WS_VISIBLE | BS_PUSHBUTTON,
			400, 700, 50, 40, hWnd, (HMENU)116, hInst, NULL);
		h_ChartRef10 = CreateWindow("button", "0.7F%", WS_CHILD | WS_VISIBLE | BS_PUSHBUTTON,
			460, 700, 50, 40, hWnd, (HMENU)117, hInst, NULL);
		h_ChartRef10 = CreateWindow("button", "0.8F%", WS_CHILD | WS_VISIBLE | BS_PUSHBUTTON,
			520, 700, 50, 40, hWnd, (HMENU)118, hInst, NULL);
		h_ChartRef10 = CreateWindow("button", "0.9F%", WS_CHILD | WS_VISIBLE | BS_PUSHBUTTON,
			580, 700, 50, 40, hWnd, (HMENU)119, hInst, NULL);
		h_ChartRef10 = CreateWindow("button", "1.0F%", WS_CHILD | WS_VISIBLE | BS_PUSHBUTTON,
			640, 700, 50, 40, hWnd, (HMENU)120, hInst, NULL);

		//MOS
		MOSedit = CreateWindow("edit", NULL, WS_CHILD | WS_VISIBLE | WS_BORDER,
			1000, 200, 100, 30, hWnd, (HMENU)201, hInst, NULL);
		//DOE
		B_DOEdistance_U = CreateWindow("button", "DOE Distance▲", WS_CHILD | WS_VISIBLE | BS_PUSHBUTTON,
			1700, 200+(70*0), 150, 30, hWnd, (HMENU)300, hInst, NULL);
		B_RotationDOE_U = CreateWindow("button", "DOE Rot▲", WS_CHILD | WS_VISIBLE | BS_PUSHBUTTON,
			1700, 200 + (70 * 1), 150, 30, hWnd, (HMENU)301, hInst, NULL);
		B_tiltXDOE_U = CreateWindow("button", "DOE Tilt X▲", WS_CHILD | WS_VISIBLE | BS_PUSHBUTTON,
			1700, 200 + (70 * 2), 150, 30, hWnd, (HMENU)302, hInst, NULL);
		B_tiltYDOE_U = CreateWindow("button", "DOE Tilt Y▲", WS_CHILD | WS_VISIBLE | BS_PUSHBUTTON,
			1700, 200 + (70 * 3), 150, 30, hWnd, (HMENU)303, hInst, NULL);
		B_sensorShiftX_U = CreateWindow("button", "Sensor Shift X▲", WS_CHILD | WS_VISIBLE | BS_PUSHBUTTON,
			1700, 200 + (70 * 4), 150, 30, hWnd, (HMENU)304, hInst, NULL);
		B_sensorShiftY_U = CreateWindow("button", "Sensor Shift Y▲", WS_CHILD | WS_VISIBLE | BS_PUSHBUTTON,
			1700, 200 + (70 * 5), 150, 30, hWnd, (HMENU)305, hInst, NULL);
		B_RxlensoffsetZ_U = CreateWindow("button", "Rx offset Z▲", WS_CHILD | WS_VISIBLE | BS_PUSHBUTTON,
			1700, 200 + (70 * 6), 150, 30, hWnd, (HMENU)306, hInst, NULL);
	//	B_RxlensShiftX_U = CreateWindow("button", "Rx shift X▲", WS_CHILD | WS_VISIBLE | BS_PUSHBUTTON,
	//		1700, 200 + (70 * 7), 150, 30, hWnd, (HMENU)307, hInst, NULL);
	//	B_RxlensShiftY_U = CreateWindow("button", "Rx shift Y▲", WS_CHILD | WS_VISIBLE | BS_PUSHBUTTON,
	//		1700, 200 + (70 * 8), 150, 30, hWnd, (HMENU)308, hInst, NULL);

		B_DOEdistance_D = CreateWindow("button", "DOE Distance ▼", WS_CHILD | WS_VISIBLE | BS_PUSHBUTTON,
			1700, 230 + (70 * 0), 150, 30, hWnd, (HMENU)310, hInst, NULL);
		B_RotationDOE_D = CreateWindow("button", "DOE Rot▼", WS_CHILD | WS_VISIBLE | BS_PUSHBUTTON,
			1700, 230 + (70 * 1), 150, 30, hWnd, (HMENU)311, hInst, NULL);
		B_tiltXDOE_D = CreateWindow("button", "DOE Tilt X▼", WS_CHILD | WS_VISIBLE | BS_PUSHBUTTON,
			1700, 230 + (70 * 2), 150, 30, hWnd, (HMENU)312, hInst, NULL);
		B_tiltYDOE_D = CreateWindow("button", "DOE Tilt Y▼", WS_CHILD | WS_VISIBLE | BS_PUSHBUTTON,
			1700, 230 + (70 * 3), 150, 30, hWnd, (HMENU)313, hInst, NULL);
		B_sensorShiftX_D = CreateWindow("button", "Sensor Shift X▼", WS_CHILD | WS_VISIBLE | BS_PUSHBUTTON,
			1700, 230 + (70 * 4), 150, 30, hWnd, (HMENU)314, hInst, NULL);
		B_sensorShiftY_D = CreateWindow("button", "Sensor Shift Y▼", WS_CHILD | WS_VISIBLE | BS_PUSHBUTTON,
			1700, 230 + (70 * 5), 150, 30, hWnd, (HMENU)315, hInst, NULL);
		B_RxlensoffsetZ_D = CreateWindow("button", "Rx offset Z▼", WS_CHILD | WS_VISIBLE | BS_PUSHBUTTON,
			1700, 230 + (70 * 6), 150, 30, hWnd, (HMENU)316, hInst, NULL);
	//	B_RxlensShiftX_D = CreateWindow("button", "Rx shift X▼", WS_CHILD | WS_VISIBLE | BS_PUSHBUTTON,
	//		1700, 230 + (70 * 7), 150, 30, hWnd, (HMENU)317, hInst, NULL);
	//	B_RxlensShiftY_D = CreateWindow("button", "Rx shift Y▼", WS_CHILD | WS_VISIBLE | BS_PUSHBUTTON,
	//		1700, 230 + (70 * 8), 150, 30, hWnd, (HMENU)318, hInst, NULL);
		B_Reset = CreateWindow("button", "Reset", WS_CHILD | WS_VISIBLE | BS_PUSHBUTTON,
			1700, 200 + (70 * 7), 150, 30, hWnd, (HMENU)319, hInst, NULL);

		DOEedit = CreateWindow("edit", NULL, WS_CHILD | WS_VISIBLE | WS_BORDER,
			1700, 170, 100, 30, hWnd, (HMENU)330, hInst, NULL);

		B_Defocus_U = CreateWindow("button", "D.F▲", WS_CHILD | WS_VISIBLE | BS_PUSHBUTTON,
			1100, 195, 40, 20, hWnd, (HMENU)320, hInst, NULL);

		B_Defocus_D = CreateWindow("button", "D.F▼", WS_CHILD | WS_VISIBLE | BS_PUSHBUTTON,
			1100, 195+20, 40, 20, hWnd, (HMENU)321, hInst, NULL);

		
	}
	case WM_TIMER:
	//	InvalidateRect(hWnd, NULL, TRUE);

		break;
	case WM_COMMAND:
	{
		switch (LOWORD(wParam))
		{
			case 201:
			{
				GetWindowText(MOSedit, cMOSedit, 256);
				MosA = atoi(cMOSedit);
				MOS_chart_distance = MosA;
				break;
			}
			case 330:
			{
				GetWindowText(DOEedit, cDOEedit, 256);
				DOEA = atoi(cDOEedit);
				DegStep = DOEA;
				break;
			}
		}
	}

	case WM_KEYDOWN:
		switch (wParam) {
		case VK_RETURN:
			break;
		case 104:
			Loaddata();
			break;
		case 121:
			ShellExecute(NULL, NULL, "input_data.ini", NULL, NULL, SW_SHOW);
			break;
		case 122:
			ShellExecute(NULL, NULL, "input_RxLens_spec.ini", NULL, NULL, SW_SHOW);
			break;
		case 123:
			ShellExecute(NULL, NULL, "input_VCSEL.ini", NULL, NULL, SW_SHOW);
			break;
		case 124:
			ShellExecute(NULL, NULL, "input_DOE.ini", NULL, NULL, SW_SHOW);
			break;
		case 101:
			Chart_Ref = 93;
			break;
		case 102:
			Chart_Ref = 50;
			break;
		case 103:
			Chart_Ref = 10;
			break;
		case VK_LEFT:
			break;
		case 105:
			if (TX_Type == 1)TX_Type = 2;
			else TX_Type = 1;
			break;		
		case 106:
		//	AreaImage();
			AreaImagei = 1;
			break;
		case 107:
			if (Outdoorsi == 0)Outdoorsi = 1;
			else Outdoorsi = 0;
			break;
		case 108:
			Save();
			break;
		case 111:
			Field = 0;
			break;
		case 112:
			Field = 1;
			break;
		case 113:
			Field = 2;
			break;
		case 114:
			Field = 3;
			break;
		case 115:
			Field = 4;
			break;
		case 116:
			Field = 5;
			break;
		case 117:
			Field = 6;
			break;
		case 118:
			Field = 7;
			break;
		case 119:
			Field = 8;
			break;
		case 120:
			Field = 9;			
			break;
		case 300:
			DOEdistance = DOEdistance + DegStep;
			break;
		case 310:
			DOEdistance = DOEdistance - DegStep;
			break;
		case 301:
			RotationDOE = RotationDOE + DegStep;
			break;
		case 311:
			RotationDOE = RotationDOE - DegStep;
			break;
		case 302:
			tiltXDOE = tiltXDOE + DegStep;
			break;
		case 312:
			tiltXDOE = tiltXDOE - DegStep;
			break;
		case 303:
			tiltYDOE = tiltYDOE + DegStep;
			break;
		case 313:
			tiltYDOE = tiltYDOE - DegStep;
			break;
		case 304:
			sensorShiftX = sensorShiftX + DegStep;
			break;
		case 314:
			sensorShiftX = sensorShiftX - DegStep;
			break;
		case 305:
			sensorShiftY = sensorShiftY + DegStep;
			break;
		case 315:
			sensorShiftY = sensorShiftY - DegStep;
			break;
		case 306:
			RxlensoffsetZ = RxlensoffsetZ + DegStep;
			break;
		case 316:
			RxlensoffsetZ = RxlensoffsetZ - DegStep;
			break;
		case 307:
			RxlensShiftX = RxlensShiftX + DegStep;
			break;
		case 317:
			RxlensShiftX = RxlensShiftX - DegStep;
			break;
		case 308:
			RxlensShiftY = RxlensShiftY + DegStep;
			break;
		case 318:
			RxlensShiftY = RxlensShiftY - DegStep;
			break;
		case 319:
			DOEdistance = 5000;
			RotationDOE = 0;
			tiltXDOE = 0;
			tiltYDOE = 0;
			sensorShiftX = 0;
			sensorShiftY = 0;
			RxlensoffsetZ = 0;
			RxlensShiftX = 0;
			RxlensShiftY = 0;
			break;
		case 320:
			if (Defocusi < 2) { Defocusi = Defocusi + 0.1; }
			break;
		case 321:
			if (Defocusi > 1) { Defocusi = Defocusi - 0.1; }
			break;
		}
		InvalidateRect(hWnd, NULL, TRUE);
		return 0;
	

	case WM_DESTROY:
		PostQuitMessage(0);

		SelectObject(h2MemDC, backBitmap);
		SelectObject(h2MemDC, hOldBitmap);
		DeleteObject(backBitmap);
		DeleteObject(hOldBitmap);
		DeleteDC(h2MemDC);

		return 0;
	case WM_PAINT:

		hdc = BeginPaint(hWnd, &ps);

		Calc_Pre();
		Calc_MOS();
		Calc_forDOE();
		

		//--------------------------------------------------------------
		//이미지 표현
		Max = -255;
		for (i = 0; i < 190; i++)
		{
			if (PreError[Field][i] > Max)Max = PreError[Field][i];
		}

		#define PreGrhX 3 //그래프 X축

		MyPen = CreatePen(PS_DOT, 1, RGB(255, 0, 0));
		OldPen = (HPEN)SelectObject(hdc, MyPen);
		if (600 - (400 / Max) > 200)
		{
			MoveToEx(hdc, 100, 600 - (400 / Max), NULL);
			LineTo(hdc, 100 + (190 * PreGrhX), 600 - (400 / Max));
			sprintf(buf, "1%%"); TextOut(hdc, 70, 600 - (400 / Max), buf, strlen(buf));
		}

		MyPen = CreatePen(PS_DOT, 0, RGB(100, 100, 100));
		OldPen = (HPEN)SelectObject(hdc, MyPen);
		MoveToEx(hdc, 10, 170, NULL);
		LineTo(hdc, 1800,170);
		MoveToEx(hdc, 10, 3, NULL);
		LineTo(hdc, 10, 170);
		MoveToEx(hdc, 10, 3, NULL);
		LineTo(hdc, 1800, 3);
		MoveToEx(hdc, 1800, 3, NULL);
		LineTo(hdc, 1800, 170);


		MyPen = CreatePen(PS_SOLID, 0, RGB(0, 0, 0));
		OldPen = (HPEN)SelectObject(hdc, MyPen);
		MoveToEx(hdc, 100, 600, NULL);
		LineTo(hdc, 100 + (190 * PreGrhX), 600);
		MoveToEx(hdc, 100, 600, NULL);
		LineTo(hdc, 100, 200);

		
		
		sprintf(buf, "%.1f%%", Max/4); TextOut(hdc, 70, 500 - 7, buf, strlen(buf));
		sprintf(buf, "%.1f%%", Max/4*2); TextOut(hdc, 70, 400 - 7, buf, strlen(buf));
		sprintf(buf, "%.1f%%", Max/4*3); TextOut(hdc, 70, 300 - 7, buf, strlen(buf));
		sprintf(buf, "%.1f%%", Max/4*4); TextOut(hdc, 70, 200 - 7, buf, strlen(buf));

		
		MyPen = CreatePen(PS_SOLID, 2, RGB(0, 0, 255));
		OldPen = (HPEN)SelectObject(hdc, MyPen);
		for (i = 0; i < 190 - 1; i++)
		{
			if (PreError[Field][i] < 1)
			{
				MyPen = CreatePen(PS_SOLID, 2, RGB(0, 0, 255));
				OldPen = (HPEN)SelectObject(hdc, MyPen);
			}
			else
			{
				MyPen = CreatePen(PS_SOLID, 2, RGB(255, 0, 0));
				OldPen = (HPEN)SelectObject(hdc, MyPen);
			}
			MoveToEx(hdc, 100 + (PreGrhX * i), 600 - (PreError[Field][i] * (400/Max)), NULL);
			LineTo(hdc, 100 + (PreGrhX * (i + 1)), 600 - (PreError[Field][i + 1] * (400 / Max)));

			SelectObject(hdc, OldPen);
			DeleteObject(MyPen);
		}

		//.................
		#define PreGrhX2 100 //그래프 X축
		#define PreGrhY2 930 //그래프 Y축

		MyPen = CreatePen(PS_DOT, 0, RGB(100, 100, 100));
		OldPen = (HPEN)SelectObject(hdc, MyPen);
		MoveToEx(hdc, PreGrhX2, PreGrhY2 -(80), NULL);	//Valid
		LineTo(hdc, PreGrhX2+190, PreGrhY2 - (80));
			MoveToEx(hdc, PreGrhX2+220, PreGrhY2 - (50), NULL);
			LineTo(hdc, PreGrhX2 + 220+190, PreGrhY2 - (50));
		MoveToEx(hdc, PreGrhX2 + 440, PreGrhY2 - (50), NULL);
		LineTo(hdc, PreGrhX2 + 440+190, PreGrhY2 - (50));

		MyPen = CreatePen(PS_SOLID, 0, RGB(0, 0, 0));
		OldPen = (HPEN)SelectObject(hdc, MyPen);
		MoveToEx(hdc, PreGrhX2, PreGrhY2, NULL);
		LineTo(hdc, PreGrhX2 + 190, PreGrhY2);
			MoveToEx(hdc, PreGrhX2 + 220, PreGrhY2, NULL);
			LineTo(hdc, PreGrhX2 + 220 + 190, PreGrhY2);
		MoveToEx(hdc, PreGrhX2 + 440, PreGrhY2, NULL);
		LineTo(hdc, PreGrhX2 + 440 + 190, PreGrhY2);

		MoveToEx(hdc, PreGrhX2, PreGrhY2-100, NULL);
		LineTo(hdc, PreGrhX2, PreGrhY2);
			MoveToEx(hdc, PreGrhX2 + 220, PreGrhY2 - 100, NULL);
			LineTo(hdc, PreGrhX2 + 220, PreGrhY2);
		MoveToEx(hdc, PreGrhX2 + 440, PreGrhY2 - 100, NULL);
		LineTo(hdc, PreGrhX2 + 440, PreGrhY2);

		//Area그래프 설명
			sprintf(buf, "Valid Ratio : %.1f M", Area_Valid_D); TextOut(hdc, PreGrhX2, PreGrhY2 -130, buf, strlen(buf));
			sprintf(buf, "Accuracy error : %.1f M", Area_Acc_D); TextOut(hdc, PreGrhX2 + 220, PreGrhY2 -130, buf, strlen(buf));
			sprintf(buf, "Precision : %.1f M", Area_Pre_D); TextOut(hdc, PreGrhX2 + 440, PreGrhY2 -130, buf, strlen(buf));

			sprintf(buf, "80%%"); TextOut(hdc, PreGrhX2-30, PreGrhY2-85, buf, strlen(buf));
			sprintf(buf, "1%%"); TextOut(hdc, PreGrhX2 - 25+220, PreGrhY2 - 60, buf, strlen(buf));
			sprintf(buf, "1%%"); TextOut(hdc, PreGrhX2 - 25+440, PreGrhY2 - 60, buf, strlen(buf));

			sprintf(buf, "distance"); TextOut(hdc, PreGrhX2 + 70, PreGrhY2, buf, strlen(buf));
			sprintf(buf, "distance"); TextOut(hdc, PreGrhX2 + 70 + 220, PreGrhY2, buf, strlen(buf));
			sprintf(buf, "distance"); TextOut(hdc, PreGrhX2 + 70 + 440, PreGrhY2, buf, strlen(buf));

		//sprintf(buf, "1%%"); TextOut(hdc, 70, 500 - 7, buf, strlen(buf));
		
		MyPen = CreatePen(PS_SOLID, 1, RGB(0, 0, 0));
		OldPen = (HPEN)SelectObject(hdc, MyPen);
		for (int i = 0; i < 190 - 1; i++)
		{
			MoveToEx(hdc, PreGrhX2 + (i), PreGrhY2 - (Area_Valid[i]), NULL);		//Val 
			LineTo(hdc, PreGrhX2 + (i + 1), PreGrhY2 - (Area_Valid[i + 1]));

			if (Area_Acc[i] < 2)
			{
				MoveToEx(hdc, PreGrhX2 + 220 + (i), PreGrhY2 - (Area_Acc[i] * 50), NULL);		//Acc
				LineTo(hdc, PreGrhX2 + 220 + (i + 1), PreGrhY2 - (Area_Acc[i + 1] * 50));
			}
			if (Area_Pre[i] < 2)
			{
				MoveToEx(hdc, PreGrhX2 + 440 + (i), PreGrhY2 - (Area_Pre[i] * 50), NULL);		//Pre
				LineTo(hdc, PreGrhX2 + 440 + (i + 1), PreGrhY2 - (Area_Pre[i + 1] * 50));
			}

		}
		
		//..................Area_Pre

		MyPen = CreatePen(PS_SOLID, 0, RGB(0, 255, 0));
		OldPen = (HPEN)SelectObject(hdc, MyPen);
		for (float i = 100; i < 100 + (190 * PreGrhX); i = i + 100)
		{
			sprintf(buf, "%.1fm", ((i - 100) / PreGrhX / 10) + 1); TextOut(hdc, i, 610, buf, strlen(buf));

		}

		//거리 판별
		for (int i = 0; i < 190; i++)
		{
			if (PreError[Field][i] > 1)
			{
				NG = i;
				break;
			}
			else { NG = 189; }
		}
		//기본조건
#define	conditionX 400
#define	conditionY 10
		sprintf(buf, "Wavelength : %.1f nm", Wavelength); TextOut(hdc, conditionX, conditionY, buf, strlen(buf));
		sprintf(buf, "OOP : %.1f W", TX_OOP); TextOut(hdc, conditionX, conditionY+20, buf, strlen(buf));
		sprintf(buf, "Frequency : %.1f nm", TX_Frq1); TextOut(hdc, conditionX, conditionY + 40, buf, strlen(buf));
		sprintf(buf, "VCSEL divergence angle : %.1f deg", VCSEL_Dangle*2); TextOut(hdc, conditionX, conditionY + 60, buf, strlen(buf));
		sprintf(buf, "VCSEL Aperturesize  : %.1f um", VCSEL_Aperturesize*1000); TextOut(hdc, conditionX, conditionY + 80, buf, strlen(buf));
		sprintf(buf, "Dot Num  : %d ea", VCSEL_emitor_Num* DOE_Num2); TextOut(hdc, conditionX, conditionY + 100, buf, strlen(buf));

		sprintf(buf, "Sensor PixelSize : %.1f um", Sensor_PixelSize); TextOut(hdc, conditionX+300, conditionY, buf, strlen(buf));
		sprintf(buf, "Exposuretime : %.1f us", Exposuretime); TextOut(hdc, conditionX+300, conditionY + 20, buf, strlen(buf));
		sprintf(buf, "Chart ref : %.1f %%", Chart_Ref); TextOut(hdc, conditionX + 300, conditionY + 40, buf, strlen(buf));
		sprintf(buf, "Tx efficiency : %.1f %%", TX_Efficiency); TextOut(hdc, conditionX + 300, conditionY + 60, buf, strlen(buf));
		sprintf(buf, "Rx efficiency : %.1f %%", Rx_lens_effiency); TextOut(hdc, conditionX + 300, conditionY + 80, buf, strlen(buf));
		sprintf(buf, "DOE efficiency : %.1f %%", DOE_efficiency); TextOut(hdc, conditionX + 300, conditionY + 100, buf, strlen(buf));

		sprintf(buf, "VCSEL : %s", Name_VCSEL); TextOut(hdc, conditionX + 600, conditionY, buf, strlen(buf));
		sprintf(buf, "DOE : %s", Name_DOE); TextOut(hdc, conditionX + 600, conditionY+20, buf, strlen(buf));
	
		sprintf(buf, "%.d mm", DOEdistance); TextOut(hdc, 1620, 220, buf, strlen(buf));
		sprintf(buf, "%.2f deg", RotationDOE); TextOut(hdc, 1620, 220 + (1 * 70), buf, strlen(buf));
		sprintf(buf, "%.2f deg", tiltXDOE); TextOut(hdc, 1620, 220 + (2 * 70), buf, strlen(buf));
		sprintf(buf, "%.2f deg", tiltYDOE); TextOut(hdc, 1620, 220 + (3 * 70), buf, strlen(buf));
		sprintf(buf, "%.2f mm", sensorShiftX); TextOut(hdc, 1620, 220 + (4 * 70), buf, strlen(buf));
		sprintf(buf, "%.2f mm", sensorShiftY); TextOut(hdc, 1620, 220 + (5 * 70), buf, strlen(buf));
		sprintf(buf, "%.2f mm", RxlensoffsetZ); TextOut(hdc, 1620, 220 + (6 * 70), buf, strlen(buf));
	//	sprintf(buf, "%.2f mm", RxlensShiftX); TextOut(hdc, 1620, 220 + (7 * 70), buf, strlen(buf));
	//	sprintf(buf, "%.2f mm", RxlensShiftY); TextOut(hdc, 1620, 220 + (8 * 70), buf, strlen(buf));
	
		HFONT hFont;
		hFont = CreateFont(30, 0, 0, 0, 0, TRUE, 0, 0, ANSI_CHARSET, 3, 2, 1,
			VARIABLE_PITCH | FF_SWISS, "Arial Black");
		SelectObject(hdc, hFont);
		
		#define	PreX 130
		#define	PreY 220
			if (TX_Type == 1) {sprintf(buf, "Flood Mode"); TextOut(hdc, PreX, PreY, buf, strlen(buf));}
			else { sprintf(buf, "Dot Mode"); TextOut(hdc, PreX, PreY, buf, strlen(buf)); }
			
			if (Outdoorsi == 0) { sprintf(buf, "Indoor"); TextOut(hdc, PreX, PreY-40, buf, strlen(buf)); }
			else { sprintf(buf, "Outdoor"); TextOut(hdc, PreX, PreY-40, buf, strlen(buf)); }

			if (NG == 189) { sprintf(buf, "성능만족 : >= 20m", Distance[NG]); TextOut(hdc, PreX, PreY+40, buf, strlen(buf)); }
			else {sprintf(buf, "성능만족 : %.1fm", Distance[NG]); TextOut(hdc, PreX, PreY+40, buf, strlen(buf));
	}
		#define	MOSX 800
		#define	MOSY 560
			if (TX_Type == 2)
			{
				sprintf(buf, "Dot크기 : %.2f mm", DotSizeAtDis); TextOut(hdc, MOSX, MOSY, buf, strlen(buf));
				sprintf(buf, "Dot Pitch : %.2f mm", MOSPitch); TextOut(hdc, MOSX, MOSY+40, buf, strlen(buf));
				sprintf(buf, "Dot/Pixel : %.2f pixel", DotSizeAtDis / PixelSizeAtDis); TextOut(hdc, MOSX, MOSY+80, buf, strlen(buf));
			}
		//
		// 
		//DOE관련
		#define DOEGrpX 1400
		#define DOEGrpY 500
		#define DOEGrpMag 80

		SensorSizeX_Half = (Sensor_PixNumH * (Sensor_PixelSize/1000))/2* DOEGrpMag;
		SensorSizeY_Half = (Sensor_PixNumV * (Sensor_PixelSize/1000))/2* DOEGrpMag;

		DOE_Count = 0;
		
		if (TX_Type == 2)
		{
			for (i = 0; i < VCSEL_emitor_Num; i++)
			{
				for (j = 0; j < DOE_Num2; j++)
				{
					if (DOEGrpX + DOE_ResultX[j][i] * DOEGrpMag > DOEGrpX - SensorSizeX_Half && DOEGrpX + DOE_ResultX[j][i] * DOEGrpMag < DOEGrpX + SensorSizeX_Half &&
						DOEGrpY + DOE_ResultY[j][i] * DOEGrpMag >DOEGrpY - SensorSizeY_Half && DOEGrpY + DOE_ResultY[j][i] * DOEGrpMag < DOEGrpY + SensorSizeY_Half)
					{
						DOE_Count++;
						MyPen = CreatePen(PS_SOLID, 3, RGB(255, 0, 0));
						OldPen = (HPEN)SelectObject(hdc, MyPen);
					}
					else
					{
						MyPen = CreatePen(PS_SOLID, 3, RGB(150, 150, 150));
						OldPen = (HPEN)SelectObject(hdc, MyPen);
					}
					MoveToEx(hdc, DOEGrpX + DOE_ResultX[j][i] * DOEGrpMag, DOEGrpY + DOE_ResultY[j][i] * DOEGrpMag, NULL);
					LineTo(hdc, DOEGrpX + DOE_ResultX[j][i] * DOEGrpMag, DOEGrpY + DOE_ResultY[j][i] * DOEGrpMag);

					SelectObject(hdc, OldPen);
					DeleteObject(MyPen);
					
				}
			}
		
		MyPen = CreatePen(PS_SOLID, 2, RGB(0, 0, 0));
		OldPen = (HPEN)SelectObject(hdc, MyPen);
		MoveToEx(hdc,	DOEGrpX-SensorSizeX_Half,		DOEGrpY- SensorSizeY_Half, NULL);
		LineTo(hdc,		DOEGrpX - SensorSizeX_Half,		DOEGrpY+ SensorSizeY_Half);
			MoveToEx(hdc, DOEGrpX + SensorSizeX_Half, DOEGrpY - SensorSizeY_Half, NULL);
			LineTo(hdc, DOEGrpX + SensorSizeX_Half, DOEGrpY + SensorSizeY_Half);
		MoveToEx(hdc, DOEGrpX - SensorSizeX_Half, DOEGrpY - SensorSizeY_Half, NULL);
		LineTo(hdc, DOEGrpX + SensorSizeX_Half, DOEGrpY - SensorSizeY_Half);
			MoveToEx(hdc, DOEGrpX - SensorSizeX_Half, DOEGrpY + SensorSizeY_Half, NULL);
			LineTo(hdc, DOEGrpX + SensorSizeX_Half, DOEGrpY + SensorSizeY_Half);

			sprintf(buf, "전체 Dot : %d EA", VCSEL_emitor_Num* DOE_Num2); TextOut(hdc, DOEGrpX - 130, DOEGrpY + 250, buf, strlen(buf));
			sprintf(buf, "유효 Dot : %.0f EA", DOE_Count); TextOut(hdc, DOEGrpX-130, DOEGrpY+280, buf, strlen(buf));
			sprintf(buf, "효율 : %.1f %%", DOE_Count/(VCSEL_emitor_Num * DOE_Num2)*100); TextOut(hdc, DOEGrpX - 130, DOEGrpY + 310, buf, strlen(buf));
		

		}
		/*
		MyPen = CreatePen(PS_SOLID, 0, RGB(0, 0, 0));
		OldPen = (HPEN)SelectObject(hdc, MyPen);
		MoveToEx(hdc, 100, 600, NULL);
		LineTo(hdc, 100 + (190 * PreGrhX), 600);
		*/
		//MOS관련
		#define MOSstartX 800
		#define MOSstartY 250
		#define MOSstartX_step 300
		#define MOSstartY_step 300

		sprintf(buf, "거리 [mm]"); TextOut(hdc, 860, 198, buf, strlen(buf));
		sprintf(buf, "Setp [deg,mm]"); TextOut(hdc, 1500, 168, buf, strlen(buf));
		
		HWND m_hWnd;

		HDC m_MemDC;
		HBITMAP m_bitmap, m_oldbitmap;

		m_hWnd = hWnd;
		hdc = GetDC(m_hWnd);
		m_MemDC = CreateCompatibleDC(hdc);

		m_bitmap = (HBITMAP)LoadImage(NULL,"human.bmp", IMAGE_BITMAP, 0, 0,LR_LOADFROMFILE);
		m_oldbitmap = (HBITMAP)SelectObject(m_MemDC, m_bitmap);
		StretchBlt(hdc, MOSstartX, MOSstartY, MOSstartX_step, MOSstartY_step, m_MemDC, 0, 0, 800, 750, SRCCOPY);
		

		MyPen = CreatePen(PS_SOLID, 1, RGB(255, 255, 0));
		OldPen = (HPEN)SelectObject(hdc, MyPen);



		if (TX_Type == 2 && MOS_chart_distance>0)
		{
			for (j = MOSstartY+10; j < MOSstartY + MOSstartY_step; j = j + (MOSPitch / 10) * 5)
			{
				if (MOSPitch < 10)break;
				for (i = MOSstartX+10; i < MOSstartX + MOSstartX_step; i = i + (MOSPitch / 10) * 5)
				{					
					EPX1 = i;
					EPY1 = j;
					EPX2 = EPX1 + (DotSizeAtDis / 10) * 5;
					EPY2 = EPY1 + (DotSizeAtDis / 10) * 5;
					Ellipse(hdc, EPX1, EPY1, EPX2, EPY2);
				}
			}
		}
				

			
		//Area image 그리기	
		hFont = CreateFont(250, 0, 0, 0, 0, TRUE, 0, 0, ANSI_CHARSET, 3, 2, 1,
			VARIABLE_PITCH | FF_SWISS, "Arial Black");
		SelectObject(hdc, hFont);
		
		if (AreaImagei == 1)
		{
			SetTextColor(hdc, RGB(255, 0, 0));
			sprintf(buf, "'Calculating...'"); TextOut(hdc, 200, 400, buf, strlen(buf));
			SetTextColor(hdc, RGB(0, 0, 0));
			InvalidateRect(hWnd, NULL, TRUE);
			AreaImage();

			h2MemDC = CreateCompatibleDC(hdc);
			backBitmap = CreateCompatibleBitmap(hdc, SensorH, SensorV); //도화지 준비!
			hOldBitmap = (HBITMAP)SelectObject(h2MemDC, backBitmap); //도화지 세팅

			for (i = 0; i < SensorH; i++)
			{
				for (j = 0; j < SensorV; j++)
				{
					int Color = EachPixel_Pre[j * SensorH + i][190] * 20000;

					MyPen = CreatePen(PS_SOLID, 2, RGB(Color, 255 - Color, 0));
					OldPen = (HPEN)SelectObject(h2MemDC, MyPen);
					MoveToEx(h2MemDC, i, j, NULL);
					LineTo(h2MemDC, i, j);
					SelectObject(h2MemDC, OldPen);
					DeleteObject(MyPen);

				}
			}
			AreaImagei = 0;
		}

		SetStretchBltMode(h2MemDC, COLORONCOLOR);
		StretchBlt(hdc, PreGrhX2 + 650, PreGrhY2 - 250, 400, 300, h2MemDC, 0, 0, SensorH, SensorV, SRCCOPY);
	
		

		//선언 제거
		SelectObject(m_MemDC, m_oldbitmap);
		DeleteDC(m_MemDC);
		DeleteObject(m_bitmap);
		DeleteObject(m_oldbitmap);;
		ReleaseDC(m_hWnd, hdc);
		SelectObject(hdc, OldPen);
		DeleteObject(MyPen);

		SelectObject(hdc, hFont);
		DeleteObject(hFont);


		EndPaint(hWnd, &ps);
		return 0;


		
	default:
		return(DefWindowProc(hWnd,iMessage,wParam,lParam));
	}
}
void Loaddata(void)
{
	char	strProfile[260];
	char	szValue[260];
	sprintf(strProfile, ".\\input_data.ini");

	::GetPrivateProfileString("General Information", "Wavelength", "0", szValue, sizeof(szValue), strProfile);
	sscanf(szValue, "%f", &Wavelength);
	::GetPrivateProfileString("General Information", "Em", "0", szValue, sizeof(szValue), strProfile);
	sscanf(szValue, "%f", &Em);
	::GetPrivateProfileString("General Information", "Spectral_Window_Min", "0", szValue, sizeof(szValue), strProfile);
	sscanf(szValue, "%f", &SWMin);
	::GetPrivateProfileString("General Information", "Spectral_Window_Max", "0", szValue, sizeof(szValue), strProfile);
	sscanf(szValue, "%f", &SWMax);
	::GetPrivateProfileString("General Information", "C", "0", szValue, sizeof(szValue), strProfile);
	sscanf(szValue, "%f", &Speedoflight);
	::GetPrivateProfileString("General Information", "IlluminationRatio", "0", szValue, sizeof(szValue), strProfile);
	sscanf(szValue, "%f", &IlluminationRatio);
	::GetPrivateProfileString("General Information", "AmbientLight_irradiance", "0", szValue, sizeof(szValue), strProfile);
	sscanf(szValue, "%f", &AmbientLight_irradiance);
	::GetPrivateProfileString("General Information", "Tx_OOP", "0", szValue, sizeof(szValue), strProfile);
	sscanf(szValue, "%f", &TX_OOP);
	::GetPrivateProfileString("General Information", "Tx_FOI_H", "0", szValue, sizeof(szValue), strProfile);
	sscanf(szValue, "%f", &TX_FOIH);
	::GetPrivateProfileString("General Information", "Tx_FOI_V", "0", szValue, sizeof(szValue), strProfile);
	sscanf(szValue, "%f", &TX_FOIV);
	::GetPrivateProfileString("General Information", "Tx_Type", "0", szValue, sizeof(szValue), strProfile);
	sscanf(szValue, "%f", &TX_Type);
	::GetPrivateProfileString("General Information", "Tx_EFL", "0", szValue, sizeof(szValue), strProfile);
	sscanf(szValue, "%f", &TX_EFL);
	::GetPrivateProfileString("General Information", "Tx_EPD_size", "0", szValue, sizeof(szValue), strProfile);
	sscanf(szValue, "%f", &TX_EPDSize);
	::GetPrivateProfileString("General Information", "Tx_EPD_Z", "0", szValue, sizeof(szValue), strProfile);
	sscanf(szValue, "%f", &TX_EPDZ);
	::GetPrivateProfileString("General Information", "Tx_CRA", "0", szValue, sizeof(szValue), strProfile);
	sscanf(szValue, "%f", &TX_CRA);
	::GetPrivateProfileString("General Information", "Tx_ModulationFrequency1", "0", szValue, sizeof(szValue), strProfile);
	sscanf(szValue, "%f", &TX_Frq1);
	::GetPrivateProfileString("General Information", "Tx_ModulationFrequency2", "0", szValue, sizeof(szValue), strProfile);
	sscanf(szValue, "%f", &TX_Frq2);
	::GetPrivateProfileString("General Information", "Tx_Efficiency", "0", szValue, sizeof(szValue), strProfile);
	sscanf(szValue, "%f", &TX_Efficiency);	


	::GetPrivateProfileString("General Information", "Sensor_Pixelsize", "0", szValue, sizeof(szValue), strProfile);
	sscanf(szValue, "%f", &Sensor_PixelSize);
	::GetPrivateProfileString("General Information", "Sensor_PixelNumH", "0", szValue, sizeof(szValue), strProfile);
	sscanf(szValue, "%f", &Sensor_PixNumH);
	::GetPrivateProfileString("General Information", "Sensor_PixelNumV", "0", szValue, sizeof(szValue), strProfile);
	sscanf(szValue, "%f", &Sensor_PixNumV);
	::GetPrivateProfileString("General Information", "Sensor_FillFactor", "0", szValue, sizeof(szValue), strProfile);
	sscanf(szValue, "%f", &Sensor_Fillfactor);
	::GetPrivateProfileString("General Information", "Sensor_Responsivity", "0", szValue, sizeof(szValue), strProfile);
	sscanf(szValue, "%f", &Sensor_Responsivity);
	::GetPrivateProfileString("General Information", "Sensor_QuantumEfficiency", "0", szValue, sizeof(szValue), strProfile);
	sscanf(szValue, "%f", &Sensor_QuantumEfficiency);	

	::GetPrivateProfileString("General Information", "Sensor_ADC_inputVrange", "0", szValue, sizeof(szValue), strProfile);
	sscanf(szValue, "%f", &Sensor_ADC_inputV);
	::GetPrivateProfileString("General Information", "Sensor_ADC_Resolution", "0", szValue, sizeof(szValue), strProfile);
	sscanf(szValue, "%f", &Sensor_ADC_Resolution);
	::GetPrivateProfileString("General Information", "Sensor_ConversionCapacitance", "0", szValue, sizeof(szValue), strProfile);
	sscanf(szValue, "%f", &Sensor_ConversionCap);
	::GetPrivateProfileString("General Information", "ofFrame", "0", szValue, sizeof(szValue), strProfile);
	sscanf(szValue, "%f", &ofFrame);
	::GetPrivateProfileString("General Information", "ExposureTime", "0", szValue, sizeof(szValue), strProfile);
	sscanf(szValue, "%f", &Exposuretime);
	::GetPrivateProfileString("General Information", "Noise_ReductionFactor", "0", szValue, sizeof(szValue), strProfile);
	sscanf(szValue, "%f", &Noise_ReductionFactor);
	::GetPrivateProfileString("General Information", "Noise_Floor", "0", szValue, sizeof(szValue), strProfile);
	sscanf(szValue, "%f", &Noise_Floor);
	::GetPrivateProfileString("General Information", "Noise_DarkSlope", "0", szValue, sizeof(szValue), strProfile);
	sscanf(szValue, "%f", &Noise_Dark);
	
	::GetPrivateProfileString("General Information", "AA_distance", "0", szValue, sizeof(szValue), strProfile);
	sscanf(szValue, "%f", &AA_distance);
	::GetPrivateProfileString("General Information", "Focusing_distance", "0", szValue, sizeof(szValue), strProfile);
	sscanf(szValue, "%f", &Focusing_distance);
	::GetPrivateProfileString("General Information", "DOE_Num", "0", szValue, sizeof(szValue), strProfile);
	sscanf(szValue, "%f", &DOE_Num);
	::GetPrivateProfileString("General Information", "Chart_Ref", "0", szValue, sizeof(szValue), strProfile);
	sscanf(szValue, "%f", &Chart_Ref);

	::GetPrivateProfileString("General Information", "RandomNoise", "0", szValue, sizeof(szValue), strProfile);
	sscanf(szValue, "%f", &RandomNoise);
	::GetPrivateProfileString("General Information", "DarkNoise", "0", szValue, sizeof(szValue), strProfile);
	sscanf(szValue, "%f", &DarkNoise);


	//VCSEL
	sprintf(strProfile, ".\\input_VCSEL.ini");
	::GetPrivateProfileString("emitor", "Comment", "0", szValue, sizeof(szValue), strProfile);
	sscanf(szValue, "%s", &Name_VCSEL);
	::GetPrivateProfileString("emitor", "Num", "0", szValue, sizeof(szValue), strProfile);
	sscanf(szValue, "%d", &VCSEL_emitor_Num);	
	::GetPrivateProfileString("emitor", "VCSEL_Dangle", "0", szValue, sizeof(szValue), strProfile);
	sscanf(szValue, "%f", &VCSEL_Dangle);
	::GetPrivateProfileString("emitor", "VCSEL_Aperturesize", "0", szValue, sizeof(szValue), strProfile);
	sscanf(szValue, "%f", &VCSEL_Aperturesize);
	for (i = 1; i < VCSEL_emitor_Num + 1; i++)
	{
		char CHAR[10];
		scanf("%d", &i);
		itoa(i, CHAR, 10);
		printf("%s", CHAR);
		LPCSTR Test = CHAR;

		::GetPrivateProfileString("x", Test, "0", szValue, sizeof(szValue), strProfile);
		sscanf(szValue, "%f", &EmitorX[i-1]);
		::GetPrivateProfileString("y", Test, "0", szValue, sizeof(szValue), strProfile);
		sscanf(szValue, "%f", &EmitorY[i - 1]);
	}
	//DOE
	::GetPrivateProfileString("emitor", "Comment", "0", szValue, sizeof(szValue), strProfile);
	sscanf(szValue, "%s", &Name_DOE);
	sprintf(strProfile, ".\\input_DOE.ini");
	::GetPrivateProfileString("DOE", "Num", "0", szValue, sizeof(szValue), strProfile);
	sscanf(szValue, "%d", &DOE_Num2);
	::GetPrivateProfileString("DOE", "PITCH", "0", szValue, sizeof(szValue), strProfile);
	sscanf(szValue, "%f", &DOE_PITCH);
	::GetPrivateProfileString("DOE", "DOE_Efficiency", "0", szValue, sizeof(szValue), strProfile);
	sscanf(szValue, "%f", &DOE_efficiency);
	::GetPrivateProfileString("DOE", "NAX", "0", szValue, sizeof(szValue), strProfile);
	sscanf(szValue, "%f", &DOE_NAX);
	::GetPrivateProfileString("DOE", "NAY", "0", szValue, sizeof(szValue), strProfile);
	sscanf(szValue, "%f", &DOE_NAY);

	i = 0;
	for (i = 1; i < DOE_Num2 + 1; i++)
	{
		char CHAR[10];
		scanf("%d", &i);
		itoa(i, CHAR, 10);
		printf("%s", CHAR);
		LPCSTR Test = CHAR;

		::GetPrivateProfileString("x", Test, "0", szValue, sizeof(szValue), strProfile);
		sscanf(szValue, "%f", &DOEX[i - 1]);
		::GetPrivateProfileString("y", Test, "0", szValue, sizeof(szValue), strProfile);
		sscanf(szValue, "%f", &DOEY[i - 1]);

		DOEX[i - 1] = DOEX[i - 1] * sin(DOE_NAX * PI / 180);
		DOEY[i - 1] = DOEY[i - 1] * sin(DOE_NAY * PI / 180);
	}	

	//Rxlens spec
	sprintf(strProfile, ".\\input_RxLens_spec.ini");

	::GetPrivateProfileString("General Information", "RX_Fno", "0", szValue, sizeof(szValue), strProfile);
	sscanf(szValue, "%f", &RX_Fno);
	::GetPrivateProfileString("General Information", "RX_FOVH", "0", szValue, sizeof(szValue), strProfile);
	sscanf(szValue, "%f", &RX_FOVH);
	::GetPrivateProfileString("General Information", "RX_FOVV", "0", szValue, sizeof(szValue), strProfile);
	sscanf(szValue, "%f", &RX_FOVV);
	::GetPrivateProfileString("General Information", "RX_EFL", "0", szValue, sizeof(szValue), strProfile);
	sscanf(szValue, "%f", &RX_EFL);
	::GetPrivateProfileString("General Information", "RX_Contrast", "0", szValue, sizeof(szValue), strProfile);
	sscanf(szValue, "%f", &RX_Contrast);
	::GetPrivateProfileString("General Information", "Rx_lens_effiency", "0", szValue, sizeof(szValue), strProfile);
	sscanf(szValue, "%f", &Rx_lens_effiency);

	for (i = 1; i < 10 + 1; i++)
	{
		char CHAR[10];
		scanf("%d", &i);
		itoa(i, CHAR, 10);
		printf("%s", CHAR);
		LPCSTR Test = CHAR;

		::GetPrivateProfileString("MTF", Test, "0", szValue, sizeof(szValue), strProfile);
		sscanf(szValue, "%f", &RxMTF[i - 1]);
		::GetPrivateProfileString("RI", Test, "0", szValue, sizeof(szValue), strProfile);
		sscanf(szValue, "%f", &RxRI[i - 1]);
	}


	TX_FOID = sqrt((Sensor_PixelSize / 1000 * Sensor_PixNumH) * (Sensor_PixelSize / 1000 * Sensor_PixNumH) + (Sensor_PixelSize / 1000 * Sensor_PixNumV) * (Sensor_PixelSize / 1000 * Sensor_PixNumV));
	TX_FOID = TX_FOID / 2;
	TX_FOID = TX_FOID / RX_EFL;
	TX_FOID = (atan(TX_FOID) / PI * 180) * 2;

	Sensor_pixelD = sqrt(Sensor_PixNumH * Sensor_PixNumH + Sensor_PixNumV * Sensor_PixNumV);
	Sensor_pixelD = Sensor_pixelD / 2;
	HalfH = Sensor_PixNumH / 2;
	HalfV = Sensor_PixNumV / 2;
	SensorH = Sensor_PixNumH;
	SensorV = Sensor_PixNumV;

}
void Calc_Pre(void)
{
	for (j = 0; j < 10; j++)
	{
		DM = 0;
		for (float i = ChartDistance_St; i < ChartDistance_ed; i = i + ChartDistance_Step)
		{
			ChartM = i;
			ChartDistance[DM] = ChartM;
			float ChartM_diff = (0.1 + 0.1 * j) * (TX_FOID / 2);
			ChartM_diff = tan(ChartM_diff * PI / 180) * i;
			ChartM_diff = sqrt(i * i + ChartM_diff * ChartM_diff);
			ChartM = ChartM_diff;

			//------------------------------------------계산 식
			//Radiance [W/m^2/sr]
			if (TX_Type == 1)
			{
				Radiance = TX_OOP / ((2 * ChartM * tan((TX_FOIH / 2) * PI / 180)) * (2 * ChartM * tan((TX_FOIV / 2) * PI / 180))) / PI;
			}
			else
			{
				Spot_EPDRatio = ((VCSEL_Dangle / TX_CRA) * TX_EPDSize) / 2;
				TX_mag = 1 / ((1 / (1 / TX_EFL - 1 / AA_distance)) / AA_distance);
				Spot_Dangle = 2 * (((atan(Spot_EPDRatio / (AA_distance + TX_EPDZ)) / PI * 180) * 0.88) + (atan((TX_mag * VCSEL_Aperturesize / 2) / (AA_distance + TX_EPDZ)) / PI * 180));
				Spot_Dangle = (Spot_Dangle / 2)*Defocusi;
				Radiance = ((TX_OOP * TX_Efficiency / 100 * DOE_efficiency / 100) / DOE_Num) / (PI * (((((tan(Spot_Dangle * PI / 180))) * (ChartM - (AA_distance / 1000))) * 1.5) * ((((tan(Spot_Dangle * PI / 180))) * (ChartM - (AA_distance / 1000))) * 1.5)));
				Radiance = Radiance / PI;


			}

			//Area[m^2]
			Rx_mag = (1 / (1 / RX_EFL - 1 / Focusing_distance)) / Focusing_distance;
			Rx_mag = 1 / Rx_mag;
			Rx_pixelDangle = atan(Rx_mag * ((Sensor_PixelSize / 2) * 0.001) / Focusing_distance);
			Rx_pixelDangle = Rx_pixelDangle / PI * 180;
			Area = (2 * atan(Rx_pixelDangle * PI / 180) * ChartM) * (2 * atan(Rx_pixelDangle * PI / 180) * ChartM);   //Precision?
		//	Area = ((2 * ChartM * tan((RX_FOVH / 2) * PI / 180)) * (2 * ChartM * tan((RX_FOVV / 2) * PI / 180)));   //Accuracy?

			//SolidAngle[sr]
			Rx_EPDsize = RX_EFL / RX_Fno;
			SolidAngle = 4 * PI * (sin((0.5 * atan(Rx_EPDsize * 0.001 / 2 / ChartM)) * (0.5 * atan(Rx_EPDsize * 0.001 / 2 / ChartM))));

			//OpticalPower[W]
			
			OpticalPower = Radiance * Area * SolidAngle * (0.01 * Chart_Ref) * (0.01 * Rx_lens_effiency * 0.01 * RxRI[j]) * (0.01 * Sensor_Fillfactor);
			signal = OpticalPower * Sensor_Responsivity * Exposuretime * 0.000001 / Em;
			shotNoise = sqrt(signal);

			OpticalPower_outdoor = AmbientLight_irradiance / PI * SolidAngle * Area * (0.01 * Chart_Ref) * (0.01 * Rx_lens_effiency * 0.01 * RxRI[j]) * (0.01 * Sensor_Fillfactor);
			signal_outdoor = OpticalPower_outdoor * Sensor_Responsivity * Exposuretime * 0.000001 / Em;
			shotNoise_outdoor = sqrt(signal+ signal_outdoor);

			if (Outdoorsi == 0)
			{
				TotalNoise = sqrt((shotNoise * shotNoise) + (RandomNoise * RandomNoise) + (DarkNoise * DarkNoise));
			}
			else
			{
				TotalNoise = sqrt((shotNoise_outdoor * shotNoise_outdoor) + (RandomNoise * RandomNoise) + (DarkNoise * DarkNoise));
			}
			//DeltaZ
			NPS = TotalNoise / signal;
			CommonCoefficient = (Speedoflight / (2 * TX_Frq1 * 1000000)) * (1 / (2 * PI)) * sqrt(2 / ofFrame) * (1 / (0.01 * RX_Contrast * 0.01 * RxMTF[j])) * (1 / Noise_ReductionFactor);
			DeltaZ = CommonCoefficient * NPS;

			Distance[DM] = i;
			//Pre[DM] = DeltaZ;
			if (DeltaZ > 0)Pre[j][DM] = DeltaZ;
			else Pre[j][DM] = 0;
			PreError[j][DM] = (Pre[j][DM] / Distance[DM]) * 100;
			DM++;
		}
	}
	
	return;
}
void Calc_MOS(void)
{
	if (TX_Type == 1 || MOS_chart_distance < 0)return;

	//MOS_chart_distance = 1; //mm
	DotSizeAtDis = 2 * tan(Spot_Dangle * PI / 180);
	DotSizeAtDis = DotSizeAtDis * MOS_chart_distance;
	MOSPitch = (atan((TX_mag * DOE_PITCH) / (AA_distance + TX_EPDZ)) / PI * 180);
	MOSPitch = tan(MOSPitch * PI / 180) * MOS_chart_distance;

	PixelSizeAtDis = (2 * atan(Rx_pixelDangle * PI / 180) * MOS_chart_distance);
	
	return;
}
void Calc_forDOE(void)
{


	for (i = 0; i < VCSEL_emitor_Num; i++)
	{
		ProcessCoordinatesX[i] = sqrt(EmitorX[i]/1000 * EmitorX[i] / 1000 + EmitorY[i] / 1000 * EmitorY[i] / 1000) * (cos(atan2(EmitorY[i] / 1000, EmitorX[i] / 1000) + (RotationDOE * PI / 180)));
		ProcessCoordinatesY[i] = sqrt(EmitorX[i] / 1000 * EmitorX[i] / 1000 + EmitorY[i] / 1000 * EmitorY[i] / 1000) * (sin(atan2(EmitorY[i] / 1000, EmitorX[i] / 1000) + (RotationDOE * PI / 180)));
	}
	for (i = 0; i < VCSEL_emitor_Num; i++)
	{
		NAProcessCoordinatesX[i] = -sin(atan(sqrt(ProcessCoordinatesX[i] * ProcessCoordinatesX[i] + ProcessCoordinatesY[i] * ProcessCoordinatesY[i]) / TX_EFL) * cos(atan2(ProcessCoordinatesY[i], ProcessCoordinatesX[i])) + (tiltXDOE * PI / 180));
		NAProcessCoordinatesY[i] = -sin(atan(sqrt(ProcessCoordinatesX[i] * ProcessCoordinatesX[i] + ProcessCoordinatesY[i] * ProcessCoordinatesY[i]) / TX_EFL) * sin(atan2(ProcessCoordinatesY[i], ProcessCoordinatesX[i])) + (tiltYDOE * PI / 180));

	}
	
	for (i = 0; i < DOE_Num2; i++)
	{
		NAProcessX[i] = sqrt(DOEX[i] * DOEX[i] + DOEY[i] * DOEY[i]) * (cos(atan2(DOEY[i], DOEX[i])));
		NAProcessY[i] = sqrt(DOEX[i] * DOEX[i] + DOEY[i] * DOEY[i]) * (sin(atan2(DOEY[i], DOEX[i])));
	}

	for(i=0; i< VCSEL_emitor_Num; i++)
	{ 
		for (j = 0; j < DOE_Num2; j++)
		{
			DOE_ResultX[j][i] = NAProcessCoordinatesX[i] + NAProcessX[j];
			DOE_ResultY[j][i] = NAProcessCoordinatesY[i] + NAProcessY[j];

			DOE_ResultX[j][i] = tan(asin(sqrt(DOE_ResultX[j][i] * DOE_ResultX[j][i] + DOE_ResultY[j][i] * DOE_ResultY[j][i]))) * cos(atan2(DOE_ResultY[j][i], DOE_ResultX[j][i]));
			DOE_ResultY[j][i] = tan(asin(sqrt(DOE_ResultX[j][i] * DOE_ResultX[j][i] + DOE_ResultY[j][i] * DOE_ResultY[j][i]))) * sin(atan2(DOE_ResultY[j][i], DOE_ResultX[j][i]));
		}
	}
	//위는 TxDOE단
	//아래부터RXDOE단
	
	
	for (i = 0; i < VCSEL_emitor_Num; i++)
	{
		for (j = 0; j < DOE_Num2; j++)
		{
			float BLx = DOE_ResultX[j][i] * DOEdistance + BaselineX + sensorShiftX;
			float BLy = DOE_ResultY[j][i] * DOEdistance + BaselineY + sensorShiftY;
			float ATYPX = atan(DOE_ResultY[j][i] / DOE_ResultX[j][i]) / PI * 180;
			float SensorX = ((RX_EFL + RxlensoffsetZ) * BLx * (-1)) / (DOEdistance + TxAssyD);
			float SensorY = ((RX_EFL + RxlensoffsetZ) * BLy * (-1)) / (DOEdistance + TxAssyD);
			float LensShiftX = DOE_ResultX[j][i] * (DOEdistance + RxAssyD) + BaselineX + (RxlensShiftX * (DOEdistance + TxAssyD)) / (RX_EFL + RxlensoffsetZ);
			float LensShiftY = DOE_ResultY[j][i] * (DOEdistance + RxAssyD) + BaselineY + (RxlensShiftY * (DOEdistance + TxAssyD)) / (RX_EFL + RxlensoffsetZ);
			float DiagonalTheta = (atan(sqrt(LensShiftX * LensShiftX + LensShiftY * LensShiftY) / (DOEdistance + TxAssyD))) / PI * 180;
			float DistoD = sqrt(SensorX * SensorX + SensorY * SensorY) / (1 - (Rxlens_distortionX3 * (DiagonalTheta * DiagonalTheta * DiagonalTheta) + Rxlens_distortionX2 * (DiagonalTheta * DiagonalTheta) + Rxlens_distortionX1 * (DiagonalTheta)+Rxlens_distortionk));
			
			if (SensorX > 0) DOE_ResultX[j][i] = DistoD * cos(ATYPX * PI / 180);
			else DOE_ResultX[j][i] = -DistoD * cos(ATYPX * PI / 180);

			if (DOE_ResultX[j][i] > 0) DOE_ResultY[j][i] = DistoD * sin(ATYPX * PI / 180);
			else DOE_ResultY[j][i] = -DistoD * sin(ATYPX * PI / 180);
		}
	}
	return;
}
double stdevRandom(double Average, double STDEV)
{
	double V1, V2, S, Temp;

	do {
		V1 = 2 * ((double)rand() / RAND_MAX) - 1;
		V2 = 2 * ((double)rand() / RAND_MAX) - 1;      // -1.0 ~ 1.0 까지의 값
		S = V1 * V1 + V2 * V2;
	} while (S >= 1 || S == 0);

	S = sqrt((-2 * log(S)) / S);
	Temp = V1 * S;
	Temp = (STDEV * Temp) + Average;

	return Temp;
}
void AreaImage(void)
{	

	for (int k = 0; k < 190; k++)
	{
		Area_Acc[k] = 0;
		Area_Pre[k] = 0;
		Area_Valid[k] = 0;

		for (i = -HalfH; i < HalfH; i++)//ChartDistance[190]
		{			
			for (j = -HalfV; j < HalfV; j++)
			{
				if (sqrt(i * i + j * j) < Sensor_pixelD * 0.1) { EachPixel_Pre[(j + HalfV) * SensorH + (i + HalfH)][k] = stdevRandom(ChartDistance[k], Pre[0][k]); }
				else if (sqrt(i * i + j * j) < Sensor_pixelD * 0.2) { EachPixel_Pre[(j + HalfV) * SensorH + (i + HalfH)][k] = stdevRandom(ChartDistance[k], Pre[1][k]); }
				else if (sqrt(i * i + j * j) < Sensor_pixelD * 0.3) { EachPixel_Pre[(j + HalfV) * SensorH + (i + HalfH)][k] = stdevRandom(ChartDistance[k], Pre[2][k]); }
				else if (sqrt(i * i + j * j) < Sensor_pixelD * 0.4) { EachPixel_Pre[(j + HalfV) * SensorH + (i + HalfH)][k] = stdevRandom(ChartDistance[k], Pre[3][k]); }
				else if (sqrt(i * i + j * j) < Sensor_pixelD * 0.5) { EachPixel_Pre[(j + HalfV) * SensorH + (i + HalfH)][k] = stdevRandom(ChartDistance[k], Pre[4][k]); }
				else if (sqrt(i * i + j * j) < Sensor_pixelD * 0.6) { EachPixel_Pre[(j + HalfV) * SensorH + (i + HalfH)][k] = stdevRandom(ChartDistance[k], Pre[5][k]); }
				else if (sqrt(i * i + j * j) < Sensor_pixelD * 0.7) { EachPixel_Pre[(j + HalfV) * SensorH + (i + HalfH)][k] = stdevRandom(ChartDistance[k], Pre[6][k]); }
				else if (sqrt(i * i + j * j) < Sensor_pixelD * 0.8) { EachPixel_Pre[(j + HalfV) * SensorH + (i + HalfH)][k] = stdevRandom(ChartDistance[k], Pre[7][k]); }
				else if (sqrt(i * i + j * j) < Sensor_pixelD * 0.9) { EachPixel_Pre[(j + HalfV) * SensorH + (i + HalfH)][k] = stdevRandom(ChartDistance[k], Pre[8][k]); }
				else { EachPixel_Pre[(j + HalfV) * SensorH + (i + HalfH)][k] = stdevRandom(ChartDistance[k], Pre[9][k]); }

				//표준편차의 제곱=분산 더하기
				Area_Acc[k] = Area_Acc[k] + sqrt((ChartDistance[k]-EachPixel_Pre[(j + HalfV) * SensorH + (i + HalfH)][k])* (ChartDistance[k] - EachPixel_Pre[(j + HalfV) * SensorH + (i + HalfH)][k]));
				//
				Area_Pre[k] = Area_Pre[k] + ((EachPixel_Pre[(j + HalfV) * SensorH + (i + HalfH)][k] - ChartDistance[k]) * (EachPixel_Pre[(j + HalfV) * SensorH + (i + HalfH)][k] - ChartDistance[k]));
				
				//3%이내값 Valid pixel로 정의
				if (
					sqrt(((ChartDistance[k]-EachPixel_Pre[(j + HalfV) * SensorH + (i + HalfH)][k]) / ChartDistance[k])* ((ChartDistance[k] - EachPixel_Pre[(j + HalfV) * SensorH + (i + HalfH)][k]) / ChartDistance[k])) < Are_Val_error
					) {Area_Valid[k] ++;	}
			}
		}
		Area_Acc[k] = (Area_Acc[k] / (Sensor_PixNumH * Sensor_PixNumV))/ ChartDistance[k] * 100;
		Area_Pre[k] = sqrt(Area_Pre[k] / (Sensor_PixNumH * Sensor_PixNumV))/ ChartDistance[k] * 100;
		Area_Valid[k] = Area_Valid[k] / (Sensor_PixNumH * Sensor_PixNumV)*100;
	}
	

	for (i = 0; i < 190; i++)
	{
		if (Area_Valid[i] < 80) { Area_Valid_D = ChartDistance[i]; break; }
	}
	for (i = 0; i < 190; i++)
	{
		if (Area_Acc[i] > 1) { Area_Acc_D = ChartDistance[i]; break; }
		//Area_Pre[i]
	}
	for (i = 0; i < 190; i++)
	{
		if (Area_Pre[i] > 1) { Area_Pre_D = ChartDistance[i]; break; }
		//Area_Pre[i]
	}
	
	/*
	char str[MAX_PATH];
	FILE* file = fopen("result.txt", "a");
	for (i = 0; i < 480; i++)
	{
		fprintf(file, "\n");
		for (j = 0; j < 640; j++)
		{
			fprintf(file, "%f ", EachPixel_Pre[j* SensorH+i][189]);
		}
	}
	*/
	return;
}
double mean(double* array, int size)
{
	double sum = 0.0;

	for (int i = 0; i < size; i++)
		sum += array[i];

	return sum / size;
}
// 표준 편차 계산 함수
double standardDeviation(double* array, int size, int option) 
{
	// 배열 요소가 1개밖에 없을 때는
	// NaN(숫자가 아님)이라는 의미로
	// -1.#IND00 을 반환
	if (size < 2) return sqrt(-1.0);

	double sum = 0.0;
	double sd = 0.0;
	double diff;
	double meanValue = mean(array, size);

	for (int i = 0; i < size; i++) {
		diff = array[i] - meanValue;
		sum += diff * diff;
	}
	sd = sqrt(sum / (size - option));

	return sd;
}
void Save(void)
{
	SYSTEMTIME time;
	GetSystemTime(&time);
	char FileName[50];
	sprintf(FileName, "Result\\Result_%d-%d-%d_%dh%dm%ds.csv", time.wYear, time.wMonth, time.wDay, time.wHour, time.wMinute, time.wSecond);
	FILE* file;
	file = fopen(FileName, "a");

	fprintf(file, "Chart Ref = %.1f%% \n", Chart_Ref);
	fprintf(file, "[MOS]\n ,거리,%.1f mm \n ,Dot크기,%.2fmm \n ,Dot pitch,%.2fmm \n ,Dot/pixel,%.2f pixel\n", MOS_chart_distance, DotSizeAtDis, MOSPitch, DotSizeAtDis / PixelSizeAtDis);
	fprintf(file, "[DOE]\n ,거리,%d mm \n ,총 Dot 수,%dEA \n ,유효 Dot 수,%.0fEA \n", DOEdistance, VCSEL_emitor_Num * DOE_Num2, DOE_Count);

	fprintf(file, "△error(%%)\n");
	fprintf(file, "Distance(m)");
	for (i = 0; i < 190; i++)
	{
		fprintf(file, ",%.1f", ChartDistance[i]);
	}fprintf(file, "\n");
	for (i = 0; i < 10; i++)
	{
		float Field2 = i;
		float Field = (Field2 + 1) / 10;
		fprintf(file, "%.1f Field", Field);
		for (j = 0; j < 190; j++)
		{
			fprintf(file, ",%.2f", PreError[i][j]);
		}
		fprintf(file, "\n");
	}

	fprintf(file, "\nPerformance of Area sensor\n");

	fprintf(file, "Distance(m)");
	for (i = 0; i < 190; i++)
	{
		fprintf(file, ",%.1f", ChartDistance[i]);
	}
	fprintf(file, "\n");
		fprintf(file, "Valid ratio(%%)");
		for (i = 0; i < 190; i++)
		{
			fprintf(file, ",%.3f", Area_Valid[i]);
		}
		fprintf(file, "\n");
	fprintf(file, "Acc error(%%)");
	for (i = 0; i < 190; i++)
	{
		fprintf(file, ",%.3f", Area_Acc[i]);
	}
	fprintf(file, "\n");
		fprintf(file, "Pre(%%)");
		for (i = 0; i < 190; i++)
		{
			fprintf(file, ",%.3f", Area_Pre[i]);
		}
		fprintf(file, "\n");
	/*
	fprintf(file, "Distance(m),Valid ratio(%%),Acc(%%),Pre(%%)\n");
	for (i = 0; i < 190; i++)
	{
		fprintf(file, "%.1f,%.3f,%.3f,%.3f\n",ChartDistance[i], Area_Valid[i], Area_Acc[i], Area_Pre[i]);
	}
	*/
	fprintf(file, "\n");


	return;
}
