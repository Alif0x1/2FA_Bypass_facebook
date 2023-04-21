import requests

def login(email, password, code=None):
    url = 'https://www.facebook.com/login.php'
    payload = {
        'email': email,
        'pass': password,
        '2fa_code': code,
    }
    session = requests.session()
    response = session.post(url, data=payload)
    if response.status_code == 200:
        return session
    else:
        return None

def bypass_two_factor(email, password):
    session = login(email, password)
    if session:
        response = session.get('https://www.facebook.com/checkpoint')
        if response.status_code == 200 and 'submit[This was me]' in response.text:
            payload = {
                'submit[This was me]': 'Continue',
            }
            response = session.post('https://www.facebook.com/checkpoint/', data=payload)
            if response.status_code == 200 and 'submit[Continue]' in response.text:
                payload = {
                    'submit[Continue]': 'Continue',
                }
                response = session.post('https://www.facebook.com/checkpoint/start/', data=payload)
                if response.status_code == 200 and 'code_input' in response.text:
                    code = input('Please enter the code sent to your phone: ')
                    payload = {
                        'code_input': code,
                        'submit[Submit Code]': 'Continue',
                    }
                    response = session.post('https://www.facebook.com/checkpoint/submit/', data=payload)
                    if response.status_code == 200 and 'checkpoint_data[submit_data][submit_type]' in response.text:
                        payload = {
                            'checkpoint_data[submit_data][submit_type]': 'continue',
                        }
                        response = session.post('https://www.facebook.com/checkpoint/submit/', data=payload)
                        if response.status_code == 200 and 'search' in response.text:
                            return session
    return None
